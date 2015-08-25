<properties
	pageTitle="DPM Protection of SharePoint farm to Azure | Microsoft Azure"
	description="This article provides an overview of DPM Protection of SharePoint farm to Azure"
	services="backup"
	documentationCenter=""
	authors="SamirMehta"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/14/2015"
	ms.author="sammehta"; "jimpark"/>


# Backing up a SharePoint farm to Azure
You back up a SharePoint farm to Azure using System Center Data Protection Manager (DPM) in much the same way you back up other data sources. Azure provides  flexibility in the backup schedule to create daily, weekly, monthly or yearly backup points and gives you varying retention policy options for various backup points. DPM provides you capability to store local disk copies for quicker recovery time objectives (RTO) and to Azure for a cost effective long term retention target.

## SharePoint supported versions and related protection scenarios
Azure Backup for DPM supports the following scenarios.

| Workload | Version | SharePoint deployment | DPM deployment type | DPM - System Center 2012 R2 | Protection and Recovery |
| -------- | ------- | --------------------- | ------------------- | --------------------------- | ----------------------- |
| SharePoint | SharePoint 2013, SharePoint 2010, SharePoint 2007, SharePoint 3.0 | SharePoint deployed as a physical server or Hyper-V/VmWare virtual machine <br> -------------- <br> SQL AlwaysOn | Physical server or on-premises Hyper-V Virtual Machine | Supports back up to Azure from Update Rollup 5 | Protect SharePoint Farm Recover: Farm, database, file or list item from disk and recovery farm and database from Azure |

## Before you start
There are a few things you need to ensure before backing up a SharePoint farm to Azure.

### Prerequisites
Before you proceed, ensure that all the  [prerequisites](backup-azure-dpm-introduction.md#prerequisites) for using Microsoft Azure Backup to protect workloads have been met. The prerequisites cover tasks such as: creating a backup vault, downloading vault credentials, installing the Azure Backup Agent and registering the server with the vault.

### DPM agent
The DPM agent must be installed on the SharePoint server, SQL servers, and any other servers that are part of the SharePoint farm. For more information about setting up the protection agent, see [Setup Protection Agent](https://technet.microsoft.com/library/hh758039.aspx). The only exception is that you only install the agent on a single Web Front End (WFE) server. DPM only needs the agent on one WFE server to serve as the entry point for protection.

### SharePoint farm
For every 10 million items in the farm, there must be at least 2 GB of space on the volume where the DPM folder is located. This space is required for catalog generation. For DPM to recover specific items (site collections, sites, lists, document libraries, folders, individual documents, and list items), catalog generation creates a list of the URLs contained within each content database. You can view the list of URLs in the recoverable item pane in the Recovery task area of DPM Administrator Console.

### SQL Server
DPM runs as a Local System and to back up SQL Server databases it needs sysadmin privileges on that account for the SQL server. Set NT AUTHORITY\SYSTEM to *sysadmin* on the SQL Server you want to back up.

In the SharePoint farm, if you have SQL Server databases that are configured with SQL Server aliases, install the SQL Server client components on the front-end Web server that DPM will protect.

### SharePoint Server
While performance depends on many factors such as size of SharePoint farm, as general guidance one DPM Server can be used to protect a 25TB SharePoint farm.

### DPM Update Rollup 5
To begin protection of a SharePoint farm to Azure, you need to install DPM Update Rollup 5 or higher. Update Rollup 5 provides the ability to protect a SharePoint farm (configured using SQL AlwaysOn) to Azure.
For more information see [installing DPM Update Rollup 5]( http://blogs.technet.com/b/dpm/archive/2015/02/11/update-rollup-5-for-system-center-2012-r2-data-protection-manager-is-now-available.aspx)

### What's not supported
1. Protecting a SharePoint farm does not protect search indexes or application service databases. You will need to configure the protection of these databases separately.
2. DPM does not provide backup of SharePoint SQL databases hosted on Scale out file server (SOFS) shares.

## Configure SharePoint Protection
You must configure the SharePoint VSS Writer service (WSS Writer service) using **ConfigureSharePoint.exe** before you can protect SharePoint using DPM.

You can find **ConfigureSharePoint.exe** in the [DPM Installation Path]\bin folder on the front-end Web server. This tool provides the protection agent with the credentials for the SharePoint farm. You run it on a single WFE server. If you have multiple WFE servers select just one when you configure a protection group.

### To configure the SharePoint VSS Writer service
1. On the WFE server, at a command prompt, go to [DPM installation location]\bin\
2. Run ConfigureSharePoint -EnableSharePointProtection
3. Enter the farm administrator credentials. This account should be a member of the local Administrator group on the WFE server. If the farm administrator isn’t a local admin grant the following permissions on the WFE server:
  - Grant the WSS_Admin_WPG group full control to the DPM folder (%Program Files%\Microsoft Data Protection Manager\DPM).
  - Grant the WSS_Admin_WPG group read access to the DPM Registry key (HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Data Protection Manager).

>[AZURE.NOTE] You’ll need to rerun ConfigureSharePoint.exe whenever there’s a change in the SharePoint farm administrator credentials.

## Back up a SharePoint farm using DPM
Once you have configured DPM and the SharePoint farm as explained above, SharePoint can be protected by DPM.

### To protect a SharePoint farm
1. From the **Protection** tab of the DPM Administrator Console, click **New**.
    ![New Protection Tab](./media/backup-azure-backup-sharepoint/dpm-new-protection-tab.png)

2. On the **Select Protection Group Type** screen of the **Create New Protection Group** wizard, select **Servers** and click **Next**.

    ![Select Protection Group type](./media/backup-azure-backup-sharepoint/select-protection-group-type.png)

3. On the **Select Group Members** screen, select the check box for the SharePoint server you want to protect and click **Next**

    ![Select group members](./media/backup-azure-backup-sharepoint/select-group-members2.png)

    >[AZURE.NOTE] With the DPM agent installed, you can see the server in the wizard. DPM also shows its structure. Because ConfigureSharePoint.exe was run, DPM communicates with the SharePoint VSS Writer and its corresponding SQL databases and recognizes the SharePoint farm structure (the associated content databases and any corresponding items).

4. On the **Select Data Protection Method** screen, enter the name of the *Protection Group* and select your preferred *protection methods*. Click **Next**.

    ![Select data protection method](./media/backup-azure-backup-sharepoint/select-data-protection-method1.png)

    >[AZURE.NOTE] Choosing disk protection helps to meet recovery time objectives with short recovery times. Azure is a cost effective long-term protection target compared to tapes. For more information see this [article](https://azure.microsoft.com/documentation/articles/backup-azure-backup-cloud-as-tape/)

5. On the **Specify Short-Term Goals** screen, select your preferred *retention range* and identify when you want backups to occur.

    ![Specify short-term goals](./media/backup-azure-backup-sharepoint/specify-short-term-goals2.png)

    >[AZURE.NOTE] Since recovery is most often required for data less than five days old, we selected a retention range of 5 days on disk and ensured that the backup happens during non-production hours, for this example.

6. Review the storage pool disk space allocated for the protection group and click **Next**.

7. For every protection group, DPM allocates disk space to store and manage replicas. At this point, DPM must create a copy of the selected data. Select how and when you want the replica created and click **Next**.

    ![Choose replica creation method](./media/backup-azure-backup-sharepoint/choose-replica-creation-method.png)

    >[AZURE.NOTE] To ensure network traffic is not effected, select a time outside production hours.

8. DPM ensures data integrity by performing consistency checks on the replica. There are two available options. You can define a schedule for running consistency checks, or allow DPM to run consistency checks automatically on the replica whenever it becomes inconsistent. Select your preferred option and click **Next**.

    ![Consistency Check](./media/backup-azure-backup-sharepoint/consistency-check.png)

9. On the **Specify Online Protection Data** screen, select the SharePoint farm you want to protect and click **Next**.

    ![DPM SharePoint Protection1](./media/backup-azure-backup-sharepoint/select-online-protection1.png)

10. On the **Specify Online Backup Schedule** screen, select your preferred schedule and click **Next**.

    ![Online_backup_schedule](./media/backup-azure-backup-sharepoint/specify-online-backup-schedule.png)

    >[AZURE.NOTE] DPM allows 2 daily backups at different times to Azure.

11. Depending on the backup schedule you selected, on the **Specify Online Retention Policy** screen, select the retention policy for daily, weekly, monthly, and yearly backup points.

    ![Online_retention_policy](./media/backup-azure-backup-sharepoint/specify-online-retention.png)

    >[AZURE.NOTE] DPM uses a Grandfather-father-son retention scheme in which a different retention policy can be chosen for different backup points.

12. Similar to disk, an initial reference point replica needs to be created in Azure. Select your preferred option for creating the initial backup copy to Azure and click **Next**.

    ![Online_replica](./media/backup-azure-backup-sharepoint/online-replication.png)

13. Review your selected settings on the **Summary** page and click **Create Group**. You will see a success message once the protection group has been created.

    ![Summary](./media/backup-azure-backup-sharepoint/summary.png)

## Restore a SharePoint item from disk using DPM
In the example below, the *Recovering SharePoint item* has been accidentally deleted and needs to be recovered.
![DPM SharePoint Protection4](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection5.png)

1. Open the **DPM Administrator Console**. All SharePoint farms protected by DPM are shown in the Protection tab.

    ![DPM SharePoint Protection3](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection4.png)

2. To begin recovering the item, select the **Recovery** tab.

    ![DPM SharePoint Protection5](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection6.png)

3. You can search SharePoint for the *Recovering SharePoint item* using a wild card based search within a recovery point range.

    ![DPM SharePoint Protection6](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection7.png)

4. Select the appropriate recovery point from the search results, right click, and select **Recover**

5. You can also navigate through various recovery points and select a database or item to recover. Select **date > Recovery time** then select the correct **database > SharePoint farm > recovery point > item**.

    ![DPM SharePoint Protection7](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection8.png)

6. Right click the item and select **Recover** to open the **Recovery Wizard**. Click **Next**.

    ![Review Recovery Selection](./media/backup-azure-backup-sharepoint/review-recovery-selection.png)

7. Select the type of recovery you want to perform and click **Next**.

    ![Recovery Type](./media/backup-azure-backup-sharepoint/select-recovery-type.png)

    >[AZURE.NOTE] Example above shows recovery item to original SharePoint site.

8. Select the **Recovery Process** you want to use.
    - Select **Recover witahout using a recovery farm** if the SharePoint farm has not changed and is the same as the recovery point that is being restored.
    - Select **Recover using a recovery farm** if the SharePoint farm has changed since the recovery point was created.

    ![Recovery Process](./media/backup-azure-backup-sharepoint/recovery-process.png)

9. Provide a staging SQL instance location for recovering the database temporarily, and provide a staging file share on the DPM Server and SharePoint server to recover the item.

    ![Staging Location1](./media/backup-azure-backup-sharepoint/staging-location1.png)

    DPM attaches the content database, which is hosting the SharePoint item, to the temporary SQL Instance. From the content database the DPM Server recovers the item and puts it on the staging file location on the DPM server. The item recovered on the staging location, on the DPM server, now needs to be exported to the staging location on the SharePoint farm.

    ![Staging Location2](./media/backup-azure-backup-sharepoint/staging-location2.png)

10. Select **Specify recovery options** and apply security settings to the SharePoint farm or apply the security settings of the recovery point. Click **Next**.

    ![Recovery Options](./media/backup-azure-backup-sharepoint/recovery-options.png)

    >[AZURE.NOTE] You can choose to throttle the network bandwidth usage. This minimizes impact to the production server during production hours.

11. Review the summary information and click **Recover** to begin recovery of the file.

    ![Recovery summary](./media/backup-azure-backup-sharepoint/recovery-summary.png)

12. Now select the **Monitoring** tab in the **DPM Administrator Console** to view the *Status* of the recovery.

    ![Recovery Status](./media/backup-azure-backup-sharepoint/recovery-monitoring.png)

    >[AZURE.NOTE] The file is now restored. You can refresh the SharePoint site to check the restored file.

## Restore a SharePoint database from Azure using DPM

1. To recover a SharePoint content database, navigate through various recovery points (as shown above) and select the recovery point to you want to recover.

    ![DPM SharePoint Protection8](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection9.png)

2. Double-click the SharePoint recovery point to show the available SharePoint catalog information.

    > [AZURE.NOTE] Since the SharePoint farm is protected for long term retention in Azure, no catalog information (meta data) is available on the DPM Server. As a result, whenever a point-in-time SharePoint content database needs to be recovered you need to re-catalog the SharePoint farm.

3. Click **Re-catalog**.

    ![DPM SharePoint Protection10](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection12.png)

    The **Cloud Recatalog** status window appears.

    ![DPM SharePoint Protection11](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection13.png)

    Once cataloging is complete, the status changes to *Success*. Click **Close**.

    ![DPM SharePoint Protection12](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection14.png)

4. Click the SharePoint object shown in DPM **Recovery** tab to get the content database structure. Right click > **Recover**.

    ![DPM SharePoint Protection13](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection15.png)

5. At this point, follow the [recovery steps above](#restore-a-sharepoint-item-from-disk-using-dpm) for recovering a Sharepoint content database from disk.

## FAQs
Q: Which versions of DPM support SQL 2014 and SQL 2012 (SP2)<br>
A: DPM 2012 R2 with Update Rollup 4 supports the same

Q: Can I recover SharePoint item to the original location if SharePoint is configured using SQL AlwaysOn (with protection on disk)?<br>
A: Yes the item can be recovered to the original SharePoint site

Q: Can I recover a SharePoint database to the original location if SharePoint is configured using SQL AlwaysOn?<br>
A: Since SharePoint databases are configured in SQL AlwaysOn, they cannot be modified unless the availability group (AG) is removed. As a result DPM cannot restore database to original location. You can recovery SQL database to another SQL instance.

## Next steps
- Learn more about DPM Protection of SharePoint - see [Video Series - DPM Protection of SharePoint](http://channel9.msdn.com/Series/Azure-Backup/Microsoft-SCDPM-Protection-of-SharePoint-1-of-2-How-to-create-a-SharePoint-Protection-Group)
- Review [Release Notes for System Center 2012 - Data Protection Manager](https://technet.microsoft.com/library/jj860415.aspx)
- Review [Release Notes for Data Protection Manager in System Center 2012 SP1](https://technet.microsoft.com/library/jj860394.aspx)
