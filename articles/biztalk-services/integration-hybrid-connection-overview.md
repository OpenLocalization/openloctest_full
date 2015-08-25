<properties 
	pageTitle="Hybrid Connections Overview | Azure" 
	description="Learn about hybrid connections, including Security, TCP Ports, and supported configurations. MABS, WABS" 
	services="biztalk-services" 
	documentationCenter="" 
	authors="MandiOhlinger" 
	manager="dwrede" 
	editor="cgronlun"/>

<tags 
	ms.service="biztalk-services" 
	ms.workload="integration" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/14/2015" 
	ms.author="mandia"/>


# Hybrid Connections Overview
This topic introduces Hybrid Connections, lists the supported configurations, and lists the required TCP Ports. 


## What is a Hybrid Connection

Hybrid Connections are a feature of Azure BizTalk Services. Hybrid Connections provide an easy and convenient way to connect Azure Web Apps (formerly websites) and Azure Mobile Apps (formerly Mobile Services) to on-premises resources behind your firewall: 

![Hybrid Connections][HCImage]

Hybrid Connections benefits include:

- Web Apps and Mobile Apps can access existing on-premises data and services securely.
- Multiple Web Apps or Mobile Apps can share a Hybrid Connection to access an on-premises resource. 
- Minimal TCP Ports are required to access your network.
- Applications using Hybrid Connections access only the specific on-premises resource that is published through the Hybrid Connection.
- Can connect to any on-premises resource that uses a static TCP port, such as SQL Server, MySQL, HTTP Web APIs, and most custom Web Services.

	> [AZURE.NOTE] TCP-based services that use dynamic ports (such as FTP Passive Mode or Extended Passive Mode) are currently not supported.

- Can be used with all frameworks supported by Azure Web Apps (.NET, PHP, Java, Python, Node.js) and Azure Mobile Apps (Node.js, .NET).
- Web Apps and Mobile Apps can access on-premises resources in exactly the same way as if the Web or Mobile App is located on your local network. For example, the same connection string used on-premises can also be used on Azure.


Hybrid Connections also provide Enterprise Administrators control and visibility into the corporate resources accessed by hybrid applications, including:

- Using Group Policy settings, Administrators can allow Hybrid Connections on the network and also designate resources that can be accessed by hybrid applications.
- Event and Audit logs on the corporate network provide visibility into the resources accessed by Hybrid Connections.


## Supported Configurations

Hybrid Connections support the following framework and application combinations:

- .NET framework access to SQL Server
- .NET framework access to HTTP/HTTPS services with WebClient
- PHP access to SQL Server, MySQL
- Java access to SQL Server, MySQL and Oracle
- Java access to HTTP/HTTPS services

When using Hybrid Connections to access on-premises SQL Server, consider the following:

- SQL Express Named Instances must be configured to use static ports. By default, SQL Express named instances use dynamic ports.
- SQL Express Default Instances use a static port, but TCP must be enabled. By default, TCP is not enabled.
- When using Clustering or Availability Groups, the `MultiSubnetFailover=true` mode is currently not supported.
- The `ApplicationIntent=ReadOnly` is currently not supported.
- SQL Authentication may be required as the end-to-end authorization method supported by the Azure application and the on-premises SQL server.


## Security and Ports

Hybrid Connections use Shared Access Signature (SAS) authorization to secure the connections from the Azure applications and the on-premises Hybrid Connection Manager to the Hybrid Connection. Separate connection keys are created for the application and the on-premises Hybrid Connection Manager. These connection keys can be rolled over and revoked independently.

Hybrid Connections provide for seamless and secure distribution of the keys to the applications and the on-premises Hybrid Connection Manager. 

See [Create and Manage Hybrid Connections](integration-hybrid-connection-create-manage.md). 

**Application authorization is separate from the Hybrid Connection**. Any appropriate authorization method can be used. The authorization method depends on the end-to-end authorization methods supported across the Azure cloud and the on-premises components. For example, your Azure application accesses an on-premises SQL Server. In this scenario, SQL Authorization may be the authorization method that is supported end-to-end.

#### TCP Ports
Hybrid Connections require only outbound TCP or HTTP connectivity from your private network. You do not need to open any firewall ports or change your network perimeter configuration to allow any inbound connectivity into your network.

The following TCP ports are used by Hybrid Connections:

Port | Why you need it
--- | ---
9350 - 9354 | These ports are used for data transmission. The Service Bus relay manager probes port 9350 to determine if TCP connectivity is available. If it is available, then it assumes that port 9352 is also available. Data traffic goes over port 9352. <br/><br/>Allow outbound connections to these ports.
5671 | When port 9352 is used for data traffic, port 5671 is used as the control channel. <br/><br/>Allow outbound connections to this port. 
80, 443 | If ports 9352 and 5671 are not usable, *then* ports 80 and 443 are the fallback ports used for data transmission and the control channel.<br/><br/>Allow outbound connections to these ports. <br/><br/>**Note** It is not recommended to use these fallback ports in place of the other TCP ports. The HTTP/WebSocket is used as the protocol instead of native TCP for data channels. It could result in lower performance.



## Next

[Create and Manage Hybrid Connections](integration-hybrid-connection-create-manage.md)<br/>
[Connect an Azure Website to an On-Premises Resource](../web-sites-hybrid-connection-get-started.md)<br/>
[Connect to on-premises SQL Server from an Azure web app](../web-sites-hybrid-connection-connect-on-premises-sql-server.md)<br/>
[Azure Mobile Services and Hybrid Connections](../mobile-services-dotnet-backend-hybrid-connections-get-started.md)


## See Also

[REST API for Managing BizTalk Services on Microsoft Azure](http://msdn.microsoft.com/library/azure/dn232347.aspx)
[BizTalk Services: Editions Chart](biztalk-editions-feature-chart.md)<br/>
[Create a BizTalk Service using Azure Management Portal](biztalk-provision-services.md)<br/>
[BizTalk Services: Dashboard, Monitor and Scale tabs](biztalk-dashboard-monitor-scale-tabs.md)<br/>

[HCImage]: ./media/integration-hybrid-connection-overview/WABS_HybridConnectionImage.png
[HybridConnectionTab]: ./media/integration-hybrid-connection-overview/WABS_HybridConnectionTab.png
[HCOnPremSetup]: ./media/integration-hybrid-connection-overview/WABS_HybridConnectionOnPremSetup.png
[HCManageConnection]: ./media/integration-hybrid-connection-overview/WABS_HybridConnectionManageConn.png 
