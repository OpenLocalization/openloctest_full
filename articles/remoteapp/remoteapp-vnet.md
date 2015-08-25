
<properties
    pageTitle="Validate the Azure VNET to use with Azure RemoteApp"
    description="Learn how to make sure your Azure VNET is ready to use with Azure RemoteApp"
    services="remoteapp"
    solutions="" documentationCenter=""
    authors="lizap"
    manager="mbaldwin" />

<tags
    ms.service="remoteapp"
    ms.workload="compute"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="06/02/2015"
    ms.author="elizapo" />



# Validate the Azure VNET to use with Azure RemoteApp

Before you use an Azure VNET with Azure RemoteApp, you might want to validate the VNET. This helps prevent issues with Azure RemoteApp connectivity.

To validate your Azure VNET, do the following:

1. Create an Azure virtual machine inside the subnet of the Azure VNET you want to use with Azure RemoteApp.

2. Connect to that VM by using the **Connect** option in the management portal.
3. Join the virtual machine to the same domain that you want to use with Azure RemoteApp.

If this is successful, the Azure VNET is ready to use with RemoteApp.

For more information about the end-to-end hybrid collection workflow, see the following articles:

- [Create a hybrid collection](remoteapp-create-hybrid-deployment.md)
- [Deploy Azure RemoteApp collection to your Azure Virtual Network (with support for ExpressRoute)](http://blogs.msdn.com/b/rds/archive/2015/04/23/deploy-azure-remoteapp-collection-to-your-azure-virtual-network-with-support-for-expressroute.aspx)
 