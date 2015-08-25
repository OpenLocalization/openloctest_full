<properties
   pageTitle="Create a load balancer using Azure Resource Manager | Microsoft Azure "
   description="How to create a load balancer rules, NAT rules, probe for Azure Resource Manager. Step by step showing end to end process to create a load balancer resource."
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="adinah"
   editor="tysonn" />
<tags
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="06/30/2015"
   ms.author="joaoma" />

# How to create a load balancer using Azure Resource Manager

The steps below will show how to create a load balancer using Azure Resource Manager with PowerShell. With Azure Resource Manager, the items to create a load balancer are configured individually and then put together to create a resource. 

We will cover in this page the sequence of individual tasks it has to be done to create a load balancer and explain in detail what is being done to accomplish the goal to create a load balancer.


## What is required to create a load balancer?

The following items need to be configured before creating a load balancer:

- Front end IP configuration 

- Backend address pool 

- Load balancing rules

- Probes 

- Inbound NAT rules 

You can get more information about load balancer components with Azure resource manager at [Azure Resource Manager support for load balancer](load-balancer-arm.md).

The following steps will show how to configure a load balancer to be load balanced between 2 virtual machines.


## Step by Step using powershell


### Create Resource Group for load balancer


### Step 1
Make sure you switch PowerShell mode to use the ARM cmdlets. More info is available at Using [Windows Powershell with Resource Manager](powershell-azure-resource-manager.md).


    PS C:\> Switch-AzureMode -Name AzureResourceManager

### Step 2

Log in to your Azure account.


    PS C:\> Add-AzureAccount

You will be prompted to Authenticate with your credentials.


### Step 3

Choose which of your Azure subscriptions to use. 

    PS C:\> Select-AzureSubscription -SubscriptionName "MySubscription"

To see a list of available subscriptions, use the ‘Get-AzureSubscription’ cmdlet.


### Step 4

Create a new resource group (skip this step if using an existing resource group)

    PS C:\> New-AzureResourceGroup -Name NRP-RG -location "West US"

Azure Resource Manager requires that all resource groups specify a location. This is used as the default location for resources in that resource group. Make sure all commands to create a load balancer will use the same resource group.

In the example above we created a resource group called "NRP-RG" and location "West US". 

## Create Virtual Network and a public IP address for front end IP pool


### Step 1

Create a virtual network:

	$backendSubnet = New-AzureVirtualNetworkSubnetConfig -Name LB-Subnet-BE -AddressPrefix 10.0.2.0/24

Creates a subnet for the virtual network and assigns to $backendSubnet

	New-AzurevirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $backendSubnet

Creates the virtual network and adds the subnet lb-subnet-be to the virtual network NRPVNet. 

### Step 2

Create a public IP address to be used by frontend IP pool:

	$publicIP = New-AzurePublicIpAddress -Name PublicIp -ResourceGroupName NRP-RG -Location "West US" –AllocationMethod Dynamic -DomainNameLabel lbip 


## Create Front end IP pool and backend address pool

Setting up a front end IP pool for the incoming load balancer network traffic and backend address pool to receive the load balanced traffic.

### Step 1 

Using public IP variable ($publicIP), create the front end IP pool.

	$frontendIP = New-AzureLoadBalancerFrontendIpConfig -Name LB-Frontend -PublicIpAddress $publicIP 


### step 2 

Set up a back end address pool used to receive incoming traffic from front end IP pool:

	$beaddresspool= New-AzureLoadBalancerBackendAddressPoolConfig -Name "LB-backend"


## Create LB rules, NAT rules, probe and load balancer

After creating the front end IP pool and the backend address pool, you will need to create the rules which will belong to the load balancer resource:

### Step 1

	$inboundNATRule1= New-AzureLoadBalancerInboundNatRuleConfig -Name "RDP1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389

	$inboundNATRule2= New-AzureLoadBalancerInboundNatRuleConfig -Name "RDP2" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3442 -BackendPort 3389

	$healthProbe = New-AzureLoadBalancerProbeConfig -Name "HealthProbe" -RequestPath "HealthProbe.aspx" -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

 	$lbrule = New-AzureLoadBalancerRuleConfig -Name "HTTP" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80


The example above is creating the following items:

- NAT rule which all incoming traffic to port 3441 will go to port 3389.
- a second NAT rule which all incoming traffic to port 3442 will go to port 3389.
- a load balancer rule which will load balance all incoming traffic on public port 80 to local port 80 in the back end address pool.
- a probe rule which will check the health status for path "HealthProbe.aspx"



### Step 2

Create the load balancer adding all objects (NAT rules, Load balancer rules, probe configurations) together:

	$NRPLB = New-AzureLoadBalancer -ResourceGroupName "NRP-RG" -Name "NRP-LB" -Location "West US" -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule1,$inboundNatRule2 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe 


## Create network interfaces

After creating the load balancer, you need define which network interfaces will be receiving the incoming load balanced network traffic, NAT rules and probe. The network interface in this case is configured individually and can be assigned to a virtual machine later on. 


### Step 1 


Get the resource virtual network and subnet to create network interfaces:

	$vnet = Get-AzureVirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG

	$backendSubnet = Get-AzureVirtualNetworkSubnetConfig -Name LB-Subnet-BE -VirtualNetwork $vnet 


In this step, we are creating a network interface which will belong to the load balancer back end pool and associate the first NAT rule for RDP for this network interface:
	
	$backendnic1= New-AzureNetworkInterface -ResourceGroupName "NRP-RG" -Name lb-nic1-be -Location "West US" -PrivateIpAddress 10.0.2.6 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[0]

### Step 2

Create a second network interface called LB-Nic2-BE:

In this step, we are creating a second network interface, assigning to the same load balancer back end pool and associating the second NAT rule created for RDP: 

 	$backendnic2= New-AzureNetworkInterface -ResourceGroupName "NRP-RG" -Name lb-nic2-be -Location "West US" -PrivateIpAddress 10.0.2.7 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[1]


The end result will show the following:


PS C:\> $backendnic1


	Name                 : lb-nic1-be
	ResourceGroupName    : NRP-RG
	Location             : westus
	Id                   : /subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/networkInterfaces/lb-nic1-be
	Etag                 : W/"d448256a-e1df-413a-9103-a137e07276d1"
	ProvisioningState    : Succeeded
	Tags                 :
	VirtualMachine       : null
	IpConfigurations     : [
                         {
                           "PrivateIpAddress": "10.0.2.6",
                           "PrivateIpAllocationMethod": "Static",
                           "Subnet": {
                             "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/virtualNetworks/NRPVNet/subnets/LB-Subnet-BE"
                           },
                           "PublicIpAddress": {
                             "Id": null
                           },
                           "LoadBalancerBackendAddressPools": [
                             {
                               "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/loadBalancers/NRPlb/backendAddressPools/LB-backend"
                             }
                           ],
                           "LoadBalancerInboundNatRules": [
                             {
                               "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/loadBalancers/NRPlb/inboundNatRules/RDP1"
                             }
                           ],
                           "ProvisioningState": "Succeeded",
                           "Name": "ipconfig1",
                           "Etag": "W/\"d448256a-e1df-413a-9103-a137e07276d1\"",
                           "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/networkInterfaces/lb-nic1-be/ipConfigurations/ipconfig1"
                         }
                       ]
	DnsSettings          : {
                         "DnsServers": [],
                         "AppliedDnsServers": []
                       }
	AppliedDnsSettings   :
	NetworkSecurityGroup : null
	Primary              : False



### Step 3 

Use the command Add-AzureVMNetworkInterface to assign the NIC to a virtual Machine.

You can find the step by step to create a virtual machine and assign to a NIC following the documentation [Create and preconfigure a Windows Virtual Machine with Resource Manager and Azure PowerShell](virtual-machines-ps-create-preconfigure-windows-resource-manager-vms.md#Example)


## See Also

[Configure a Load balancer distribution mode](load-balancer-distribution-mode.md)

[Configure idle TCP timeout settings for your load balancer](load-balancer-tcp-idle-timeout.md)
 
