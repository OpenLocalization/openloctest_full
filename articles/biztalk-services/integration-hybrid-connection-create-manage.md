<properties 
	pageTitle="Create and Manage Hybrid Connections | Azure" 
	description="Learn how to create a hybrid connection, manage the connection, and install the Hybrid Connection Manager. MABS, WABS" 
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
	ms.date="07/14/2015" 
	ms.author="mandia"/>


# Create and Manage Hybrid Connections


## Overview of the Steps
1. Create a Hybrid Connection by entering the host name or IP address of the on-premises resource in your private network.
2. Link your Azure Web Apps or Azure Mobile Apps to the Hybrid Connection.
3. Install the Hybrid Connection Manager on your on-premises resource and connect to the specific Hybrid Connection. The Azure portal provides a single-click experience to install and connect.
4. Manage Hybrid Connections and their connection keys.

This topic lists these steps. 


## <a name="CreateHybridConnection"></a>Create a Hybrid Connection

A Hybrid Connection can be created in the Azure Management Portal using Web Apps **or** using BizTalk Services. 

**To create Hybrid Connections using Web Apps**, see [Connect Azure Web Apps to an On-Premises Resource](../web-sites-hybrid-connection-get-started.md).

**To create Hybrid Connections in BizTalk Services**:

1. Sign in to the [Azure Management Portal](http://go.microsoft.com/fwlink/p/?LinkID=213885).
2. In the left navigation pane, select **BizTalk Services** and then select your BizTalk Service. 
<br/>If you don't have an existing BizTalk Service, you can [Create a BizTalk Service](biztalk-provision-services.md).
3. Select the Hybrid Connections tab:
<br/>
![Hybrid Connections Tab][HybridConnectionTab]

4. Select **Create a Hybrid Connection** or select the **ADD** button in the task bar. Enter the following:

	Property | Description
--- | ---
Name | The Hybrid Connection name must be unique and cannot be the same name as the BizTalk Service. You can enter any name but be specific with its purpose. Examples include:<br/><br/>Payroll*SQLServer*<br/>SupplyList*SharepointServer*<br/>Customers*OracleServer*
Host Name | Enter the fully qualified host name, only the host name, or the IPv4 address of the on-premises resource. Examples include:<br/><br/>mySQLServer<br/>*mySQLServer*.*Domain*.corp.*yourCompany*.com<br/>*myHTTPSharePointServer*<br/>*myHTTPSharePointServer*.*yourCompany*.com<br/>10.100.10.10
Port | Enter the port number of the on-premises resource. For example, if you're using Web Apps, enter port 80 or port 443. If you're using SQL Server, enter port 1433.

5. Select the check mark to complete the setup. 

#### Additional

- Multiple Hybrid Connections can be created. See the [BizTalk Services: Editions Chart](biztalk-editions-feature-chart.md) for the number of connections allowed. 
- Each Hybrid Connection is created with a pair of connection strings: Application keys that SEND and On-premises keys that LISTEN. Each pair has a Primary and a Secondary key. 


## <a name="LinkWebSite"></a>Link your Azure Web Apps or Azure Mobile Apps

To link the Azure Web Apps to an existing Hybrid Connection, select **use an existing Hybrid Connection** in the Hybrid Connections blade. See [Connect Azure Web Apps to an On-Premises Resource](../web-sites-hybrid-connection-get-started.md).

To link the Azure Mobile Apps to an existing Hybrid Connection, select **add hybrid connection** when changing or creating a Mobile Service. See [Azure Mobile Services and Hybrid Connections](../mobile-services-dotnet-backend-hybrid-connections-get-started.md).


## <a name="InstallHCM"></a>Install the Hybrid Connection Manager on-premises

After a Hybrid Connection is created, install the Hybrid Connection Manager on the on-premises resource. It can be downloaded from your Azure Web Apps or from your BizTalk Service. BizTalk Services steps: 

1. Sign in to the [Azure Management Portal](http://go.microsoft.com/fwlink/p/?LinkID=213885).
2. In the left navigation pane, select **BizTalk Services** and then select your BizTalk Service. 
3. Select the **Hybrid Connections** tab:
<br/>
![Hybrid Connections Tab][HybridConnectionTab]
4. In the task bar, select **On-Premises Setup**:
<br/>
![On-Premises Setup][HCOnPremSetup]
5. Select **Install and Configure** to run or download the Hybrid Connection Manager on the on-premises system. 
6. Select the check mark to start the installation. 

<!--
You can also download the Hybrid Connection Manager MSI file and copy the file to your on-premises resource. Specific steps:

1. Copy the on-premises primary Connection String. See [Manage Hybrid Connections](#ManageHybridConnection) in this topic for the specific steps.
2. Download the Hybrid Connection Manager MSI file. 
3. On the on-premises resource, install the Hybrid Connection Manager from the MSI file. 
4. Using Windows PowerShell, type: 
> Add-HybridConnection -ConnectionString “*Your On-Premises Connection String that you copied*” 
--> 

#### Additional
- Hybrid Connections support on-premises resources installed on the following operating systems:

	- Windows Server 2008 R2
	- Windows Server 2012
	- Windows Server 2012 R2


- After you install the Hybrid Connection Manager, the following occurs: 

	- The Hybrid Connection hosted on Azure is automatically configured to use the Primary Application Connection String. 
	- The On-Premises resource is automatically configured to use the Primary On-Premises Connection String.

- The Hybrid Connection Manager must use a valid on-premises connection string for authorization. The Azure Web Apps or Mobile Apps must use a valid application connection string for authorization.
- You can scale Hybrid Connections by installing another instance of the Hybrid Connection Manager on another server. Configure the on-premises listener to use the same address as the first on-premises listener. In this situation, the traffic is randomly distributed (round robin) between the active on-premises listeners. 


## <a name="ManageHybridConnection"></a>Manage Hybrid Connections
To manage your Hybrid Connections, you can:

- Use the Azure portal and go to your BizTalk Service. 
- Use [REST APIs](http://msdn.microsoft.com/library/azure/dn232347.aspx).

#### Copy/regenerate the Hybrid Connection Strings

1. Sign in to the [Azure Management Portal](http://go.microsoft.com/fwlink/p/?LinkID=213885).
2. In the left navigation pane, select **BizTalk Services** and then select your BizTalk Service. 
3. Select the **Hybrid Connections** tab:
<br/>
![Hybrid Connections Tab][HybridConnectionTab]
4. Select the Hybrid Connection. In the task bar, select **Manage Connection**:
<br/>
![Manage Options][HCManageConnection]
<br/>
**Manage Connection** lists the Application and On-Premises connection strings. You can copy the Connection Strings or regenerate the Access Key used in the connection string. 
<br/>
<br/>
**If you select Regenerate**, the Shared Access Key used within the Connection String is changed. Do the following:
- In the Azure Management Portal, select **Sync Keys** in the Azure application.
- Re-run the **On-Premises Setup**. When you re-run the On-Premises Setup, the on-premises resource is automatically configured to use the updated Primary connection string.


#### Use Group Policy to control the on-premises resources used by a Hybrid Connection

1. Download the [Hybrid Connection Manager Administrative Templates](http://www.microsoft.com/download/details.aspx?id=42963).
2. Extract the files.
3. On the computer that modifies group policy, do the following: 

	- Copy the .ADMX files to the *%WINROOT%\PolicyDefinitions* folder.
	- Copy the .ADML files to the *%WINROOT%\PolicyDefinitions\en-us* folder.

Once copied, you can use Group Policy Editor to change the policy.




## Next

[Connect Azure Web Apps to an On-Premises Resource](../web-sites-hybrid-connection-get-started.md)<br/>
[Connect to on-premises SQL Server from Azure Web Apps](../web-sites-hybrid-connection-connect-on-premises-sql-server.md)<br/>
[Azure Mobile Services and Hybrid Connections](../mobile-services-dotnet-backend-hybrid-connections-get-started.md)<br/>
[Hybrid Connections Overview](integration-hybrid-connection-overview.md)


## See Also

[REST API for Managing BizTalk Services on Microsoft Azure](http://msdn.microsoft.com/library/azure/dn232347.aspx)<br/>
[BizTalk Services: Editions Chart](biztalk-editions-feature-chart.md)<br/>
[Create a BizTalk Service using Azure Management Portal](biztalk-provision-services.md)<br/>
[BizTalk Services: Dashboard, Monitor and Scale tabs](biztalk-dashboard-monitor-scale-tabs.md)<br/>


[HybridConnectionTab]: ./media/integration-hybrid-connection-create-manage/WABS_HybridConnectionTab.png
[HCOnPremSetup]: ./media/integration-hybrid-connection-create-manage/WABS_HybridConnectionOnPremSetup.png
[HCManageConnection]: ./media/integration-hybrid-connection-create-manage/WABS_HybridConnectionManageConn.png 