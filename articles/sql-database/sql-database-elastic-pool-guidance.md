<properties 
	pageTitle="Price and performance considerations for an elastic database pool | Azure SQL Database elastic database pools" 
	description="An elastic database pool is a collection of available resources that are shared by a group of elastic databases. This document provides guidance to help assess the suitability of using an elastic database pool for a group of databases." 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database"
	ms.devlang="NA"
	ms.date="06/24/2015" 
	ms.author="sstein" 
	ms.workload="data-management" 
	ms.topic="article" 
	ms.tgt_pltfrm="NA"/>


# Price and performance considerations for an elastic database pool


This document provides guidance to help assess if using an elastic database pool for a group of databases is cost efficient based on database usage patterns and pricing differences between an elastic pool and single databases. Additional guidance is also provided to assist in determining the current pool size required for an existing set of SQL databases.  

- For an overview of elastic database pools, see [SQL Database elastic database pools](sql-database-elastic-pool.md).
- For detailed information about elastic database pools, see [SQL Database elastic database pool reference](sql-database-elastic-pool-reference.md).


> [AZURE.NOTE] Elastic pools are currently in preview, and only available with SQL Database V12 Servers.

## Elastic database pools

SaaS ISVs develop applications built on top of large scale data-tiers consisting of multiple databases. A common application pattern is for each of these databases to have different customers with uniquely varying and unpredictable usage patterns. It can be difficult for the ISV to predict the resource requirements of each database individually. Under these circumstances, the ISV can overprovision resources at considerable expense to ensure favorable throughput and response times for all databases. Or, the ISV can spend less and risk a poor performance experience for their customers.  

Elastic database pools in Azure SQL Database enable SaaS ISVs to optimize the price performance for a group of databases within a prescribed budget while delivering performance elasticity for each database. Elastic pools enable the ISV to purchase database throughput units (DTUs) for an elastic pool shared by multiple databases to accommodate unpredictable periods of usage by individual databases. The DTU requirement for an elastic pool is determined by the aggregate utilization of its databases. The amount of DTUs available to the elastic pool is controlled by the ISV budget. Elastic pools makes it easy for the ISV to reason over the impact of budget on performance and vice versa for their pool. The ISV simply adds databases to the elastic pool, sets any required DTU guarantees or caps for the databases, and then sets the DTU of the pool based on their budget. By using elastic pools ISVs can seamlessly grow their service from a lean startup to a mature business at ever increasing scale.  
  


## When to consider an elastic database pool

Elastic pools are well suited for a large number of databases with specific utilization patterns. For a given database, this pattern is characterized by low average utilization with relatively infrequent utilization spikes.

The more databases you can add to a pool the greater your savings become, but depending on your application utilization pattern, it is possible to see savings with as few as 4 S3 databases.  

The following sections will help you understand how to assess if your specific collection of databases will benefit from using an elastic pool.

### Assessing database utilization patterns

The following figure shows an example of a database that spends much time idle, but also periodically spikes with activity. This is a utilization pattern that is well suited for an elastic pool: 
 
   ![one database][1]

For the one hour period illustrated above, DB1 peaks up to 90 DTUs, but its overall average usage is <5 DTUs. An S3 performance level is required to run this workload in a single database. This however leaves most of the resources unused during periods of low activity. 

An elastic pool allows these unused DTUs to be shared across multiple databases, and so reduces the total amount of DTUs needed and overall cost.

Building on the previous example, suppose there are additional databases with similar utilization patterns as DB1. In the next two figures below, the utilization of 4 databases and 20 databases are layered onto the same graph to illustrate the non-overlapping nature of their utilization over time: 

   ![four database][2]

   ![twenty databases][3]

The aggregate DTU utilization across all 20 databases is illustrated by the black line in the above figure. This shows that the aggregate DTU utilization never exceeds 100 DTUs, and indicates that the 20 databases can share 100 DTUs over this time period. This results in a 20x reduction in DTUs and a 6x price reduction compared to placing each of the databases in S3 performance levels for single databases. 


This example is ideal for the following reasons: 

- There are large differences between peak utilization and average utilization per database.  
- The peak utilization for each database occurs at different points in time. 
- DTUs are shared between a large number of databases.


The price for an elastic pool is a function of the pool DTUs and the number of databases within it. While the DTU unit price for a pool at GA pricing is 3x greater than the DTU unit price for a single database, pool DTUs can be shared by many databases and so in many cases fewer total DTUs are needed. These distinctions in pricing and DTU sharing are the basis of the price savings potential that pools can provide.  

<br>

The following rules of thumb related to database count and database utilization help to ensure that an elastic pool delivers reduced cost compared to using performance levels for single databases. The guidance is based on general availability (GA) prices. Note that GA prices are discounted by 50% during the preview, and so these rules of thumb should be considered relatively conservative. 


### Minimum number of databases

With GA pricing, an elastic pool becomes more of a cost effective performance choice if 1 DTU can be shared by more than 3 databases. This means that the sum of the DTUs of performance levels for single databases is more than 3x the DTUs of the pool. For available sizes, see [DTU and storage limits for elastic pools and elastic databases](sql-database-elastic-pool-reference.md#dtu-and-storage-limits-for-elastic-pools-and-elastic-databases).


***Example***<br>
At least 4 S3 databases or at least 36 S0 databases are needed for a 100 DTU elastic pool to be more cost efficient than using performance levels for single databases. (Note that with preview prices, the pricing breakeven point based on database count lowers to 2 S3 databases or 17 S0 databases.)



### Maximum number of concurrently peaking databases

By sharing DTUs, not all databases in a pool can simultaneously use DTUs up to the limit available when using performance levels for single databases. The fewer databases that concurrently peak, the lower the pool DTU can be set and the more cost efficient it becomes. In general, not more than 1/3 of the databases in the pool should simultaneously peak to their DTU limit. 

***Example***<br>
To reduce costs for 4 S3 databases in a 200 DTU pool, at most 2 of these databases can simultaneously peak in their utilization.  Otherwise, if more than 2 of these 4 S3 databases simultaneously peak, the pool would have to be sized to more than 200 DTUs.  And if the pool is resized to more than 200 DTUs, more S3 databases would need to be added to the pool to keep costs lower than performance levels for single databases.  


Note this example does not consider utilization of other databases in the pool. If all databases have some utilization at any given point in time, then less than 1/3 of the databases can peak simultaneously. 


### DTU utilization per database

A large difference between the peak and average utilization of a database indicates prolonged periods of low utilization and short periods of high utilization. This utilization pattern is ideal for sharing resources across databases. A database should be considered for a pool when its peak utilization is about 3 times greater than its average utilization. 

    
***Example***<br>
An S3 database that peaks to 100 DTUs and on average uses 30 DTUs or less is a good candidate for sharing DTUs in an elastic pool.  Alternatively, an S1 database that peaks to 20 DTUs and on average uses 7 DTUs or less is a good candidate for an elastic pool. 
    

## Heuristic to compare the pricing difference between an elastic pool and single databases 

The following heuristic can help estimate whether an elastic pool is more cost effective than using individual single databases.

1. Estimate the DTUs needed for the pool by the following:
    
    MAX(*Total number of DBs* * *average DTU utilization per DB*, *number of concurrently peaking DBs* * *Peak DTU utilization per DB*)

2. Select the smallest available DTU value for the pool that is greater than the estimate from Step 1. For available DTU choices, see the valid values for DTUs listed here: [DTU and storage limits for elastic pools and elastic databases](sql-database-elastic-pool-reference.md#dtu-and-storage-limits-for-elastic-pools-and-elastic-databases).

    


3. Calculate the price for the pool as follows:

    pool price = (*pool DTUs* * *pool DTU unit price*) + (*total number DBs* * *pool DB unit price*)

    See [SQL Database Pricing](http://azure.microsoft.com/pricing/details/sql-database/) for pricing information.   


4. Compare the pool price from Step 3 to the price of using the appropriate performance levels for single databases. 



## Determining the best pool DTU size for existing SQL Databases 

The best size for an elastic pool depends on the aggregate DTUs and storage resources needed for all databases in the pool. This involves determining the larger of the following two quantities: 

* Maximum DTUs utilized by all databases in the pool.
* Maximum storage bytes utilized by all databases in the pool. 

Note that for the Standard service tier, 1 GB of storage is allowed for every 1 DTU configured for the pool. For example, if a pool is configured with 200 DTUs, then its storage limit is 200 GB.    

### Use Service Tier Advisor (STA) and Dynamic Management Views (DMVs) for sizing recommendations   

STA and DMVs provide different tooling options and capabilities for sizing an elastic pool. Regardless of the tooling option used, the sizing estimate should only be used for initial assessment and creation of elastic pools. Once an elastic pool is created, its resource usage should be accurately monitored and the performance settings of the pool adjusted up and down as needed. 

**STA**<br>STA is a built-in tool in the Azure portal that automatically evaluates historical resource utilization of databases in an existing SQL Database server and recommends an appropriate elastic pool configuration.

**DMV sizing tool**<br>DMV sizing tool is provided as a PowerShell script and enables customizing the sizing estimates of an elastic pool for existing databases in a server. 

### Choosing between STA and DMV tooling 

Select the tool that is appropriate for analyzing your specific application. The following table summarizes the differences between these 2 sizing tools:

| Capability | STA | DMVs |
| :--- | :--- | :--- |
| Granularity of data samples used in the analysis.  | 15 seconds | 15 seconds |
| Considers pricing differences between a pool and performance levels for single databases. | Yes | No |
| Allows customizing the list of the databases analyzed within a server. | No | Yes |
| Allows customizing the period of time used in the analysis. | No | Yes |


### Estimating elastic pool size using STA  

STA evaluates the utilization history of databases and recommends an elastic pool when it is more cost effective than using performance levels for single databases. If an elastic pool is recommended, the tool provides a list of recommended databases, and also the recommended amount of pool DTUs and min/max DTU settings for each elastic database. In order for a database to be considered as a candidate for an elastic pool, it must exist for at least 14 days. The elastic database pool public preview is limited to Standard only, so Premium databases are not yet considered as candidates for an elastic pool.

STA is available in the Azure portal when adding an elastic pool to an existing server. If recommendations for an elastic pool are available for that server, they are displayed in the “Elastic Database Pool’ creation blade. Customers can always change the recommended configurations to create their own elastic pool grouping. 

### Estimating elastic pool size using Dynamic Management Views (DMVs) 

The [sys.dm_db_resource_stats](https://msdn.microsoft.com/library/dn800981.aspx) DMV measures resource utilization of an individual database. This DMV provides CPU, IO, log, and log utilization of a database as a percentage of the database's performance level limit. This data can be used to calculate the DTU utilization of a database in any given 15 second interval. 

The aggregate DTU utilization for an elastic pool in a 15 second interval can be estimated by aggregating the DTU utilization for all candidate databases during that time. Depending on the specific performance goals, it may make sense to discard a small percentage of the sample data. For example, a 99th percentile value of aggregate DTUs across all time intervals can be applied to exclude outliers and provide an elastic pool DTU to fit 99% of the sampled time intervals. 

## PowerShell script for estimating your databases aggregate DTU usage

A sample PowerShell script is provided here to estimate the aggregate DTU values for user databases in a SQL Database server.

The script only collects data while it is running. For a typical production workload you should run the script for at least a day, although a week or even longer will likely give a more accurate estimate. Run the script for a duration of time that represents your databases typical workload.

> [AZURE.IMPORTANT] You must keep the PowerShell window open while running the script. Do not close the PowerShell window until you have run the script for a sufficient amount of time and captured enough data to represent your typical workload spanning both normal and peak usage times. 

### Script prerequisites 

Install the following prior to running the script.:

- The latest [Powershell command-line tools](http://go.microsoft.com/?linkid=9811175&clcid=0x409).
- The [SQL Server 2014 feature pack](https://www.microsoft.com/download/details.aspx?id=42295).


### Script details


You can run the script from your local machine or a VM on the cloud. When running it from your local machine, you may incur data egress charges because the script needs to download data from your target databases. Below shows data volume estimation based on number of target databases and duration of running the script. For Azure data transfer costs refer to [Data Transfer Pricing Details](http://azure.microsoft.com/pricing/details/data-transfers/).
       
 -     1 database per hour = 38KB
 -     1 database per day = 900KB
 -     1 database per week = 6MB
 -     100 databases per day = 90MB
 -     500 databases per week = 3GB

The script excludes certain databases that are not good candidates for the current public preview offering of the Standard Elastic Pool tier. 
If you need to exclude additional databases from the target server, you may change the script to meet your criteria. By default, the script does not compile information for the following:

* All Premium databases on V12 servers.
* P2 and P3 databases on V11 servers.
* Elastic databases (databases already in an elastic pool).
* The server's master database.

The script needs an output database to store intermediate data for analysis. You can use a new or existing database. Although not technically required for the tool to run, the output database should be in a different server to avoid impacting the analysis outcome. Suggest the performance level of the output database to be at least S0 or higher. When collecting a long duration of data for a large number of databases, you may consider upgrading your output database to a higher performance level.

The script needs you to provide the credentials to connect to the target server (the elastic database pool candidate) with full server name like “abcdef.database.windows.net”. Currently the script doesn’t support analyzing multiple servers at a time.



After submitting values for the initial set of parameters, you are prompted to log on your azure account. This is for logging on to your target server, not the output database server.
	
If you run into the following warnings while running the script you can ignore them:

- WARNING: The Switch-AzureMode cmdlet is deprecated.
- WARNING: Could not obtain SQL Server Service information. An attempt to connect to WMI on 'Microsoft.Azure.Commands.Sql.dll' failed with the following error: The RPC server is unavailable.

When the script completes it will output the estimated number of DTUs required for an elastic pool to contain all candidate databases in the target server. This estimated DTU can be used for creating and configuring an elastic pool to hold these databases. Once the pool is created and databases moved into the pool, it should be monitored closely for a few days and any adjustments to the pool DTU configuration should be made as needed.


To select the entire script for copying, click any text in the script 3 times (triple-click).

    
    param (
    [Parameter(Mandatory=$true)][string]$AzureSubscriptionName, # Azure Subscription name - can be found on the Azure portal: https://portal.azure.com/
    [Parameter(Mandatory=$true)][string]$ResourceGroupName, # Resource Group name - can be found on the Azure portal: https://portal.azure.com/
    [Parameter(Mandatory=$true)][string]$servername, # full server name like "abcdefg.database.windows.net"
    [Parameter(Mandatory=$true)][string]$username, # user name
    [Parameter(Mandatory=$true)][string]$serverPassword, # password
    [Parameter(Mandatory=$true)][string]$outputServerName, # metrics collection database for analysis. full server name like "zyxwvu.database.windows.net"
    [Parameter(Mandatory=$true)][string]$outputdatabaseName,
    [Parameter(Mandatory=$true)][string]$outputDBUsername,
    [Parameter(Mandatory=$true)][string]$outputDBpassword,
    [Parameter(Mandatory=$true)][int]$duration_minutes # How long to run. Recommend to run for the period of time when your typical workload is running. At least 10 mins.
    )
    
    Add-AzureAccount 
    Select-AzureSubscription $AzureSubscriptionName
    Switch-AzureMode AzureResourceManager
    
    $server = Get-AzureSqlServer -ServerName $servername.Split('.')[0] -ResourceGroupName $ResourceGroupName
    
    # Check version/upgrade status of the server
    $upgradestatus = Get-AzureSqlServerUpgrade -ServerName $servername.Split('.')[0] -ResourceGroupName $ResourceGroupName
    $version = ""
    if ([string]::IsNullOrWhiteSpace($server.ServerVersion)) 
    {
    $version = $upgradestatus.Status
    }
    else
    {
    $version = $server.ServerVersion
    }
    
    # For Elastic database pool candidates, we exclude master, Premium SKU (P1 in previous version still qualifies), and any databases that are already in a pool. You may add more to the excluded list below as needed
    if ($version -in ("12.0", "Completed")) # This is on Azure Database V12. 
    {
    $ListOfDBs = Get-AzureSqlDatabase -ServerName $servername.Split('.')[0] -ResourceGroupName $ResourceGroupName | Where-Object {$_.DatabaseName -notin ("master") -and $_.CurrentServiceLevelObjectiveName -notin ("ElasticPool") -and $_.CurrentServiceLevelObjectiveName -notlike ("P*") -and $_.CurrentServiceObjectiveName -notin ("ElasticPool") -and $_.CurrentServiceObjectiveName -notlike ("P*")}
    }
    else # This is previous versions of Azure Database
    {
    $ListOfDBs = Get-AzureSqlDatabase -ServerName $servername.Split('.')[0] -ResourceGroupName $ResourceGroupName | Where-Object {$_.DatabaseName -notin ("master") -and $_.CurrentServiceLevelObjectiveName -notin ("ElasticPool", "P2", "P3") -and $_.CurrentServiceObjectiveName -notin ("ElasticPool", "P2", "P3")}
    }
    
    # Due to pricing model difference, we don't recommend to put less than 3 databases in a pool. 
    if ($ListOfDBs.Count -lt 3)
    {
    Write-Host "There are less than 3 qualified standalone databases in this server. Not a good candidate for Elastic Database Pool!" -ForegroundColor Red
    }
    
    $outputConnectionString = "Data Source=$outputServerName;Integrated Security=false;Initial Catalog=$outputdatabaseName;User Id=$outputDBUsername;Password=$outputDBpassword"
    $destinationTableName = "resource_stats_output"
    
    # Create a table in output database for metrics collection
    $sql = "
    IF  NOT EXISTS (SELECT * FROM sys.objects 
    WHERE object_id = OBJECT_ID(N'$($destinationTableName)') AND type in (N'U'))
    
    BEGIN
    Create Table $($destinationTableName) (database_name varchar(128), end_time datetime, avg_cpu float, avg_io float, avg_log float, db_size float);
    Create Clustered Index ci_endtime ON $($destinationTableName) (end_time);
    END
    TRUNCATE TABLE $($destinationTableName);
    "
    Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $sql -ConnectionTimeout 120 -QueryTimeout 120 
    
    # waittime (minutes) is interval between data collection queries in the loop below.
    $Waittime = 10
    $end_Time = [DateTime]::UtcNow
    $start_time = $end_time.AddMinutes(-$Waittime)
    $finish_time = $end_Time.AddMinutes($duration_minutes)
    
    While ($end_time -lt $finish_time)
    {
     foreach ($db in $ListOfDBs)
     {
    if ($version -in ("12.0", "Completed")) # for V12 databases 
    {
    $sql = "Declare @dbname varchar(128) = '$($db.DatabaseName)';"
    $sql += "Declare @SLO varchar(20) = '$($db.CurrentServiceLevelObjectiveName)';"
    $sql+= "
    Declare @DTU_cap int, @db_size float;
    Select @DTU_cap = CASE @SLO 
    WHEN 'Basic' THEN 5
    WHEN 'S0' THEN 10
    WHEN 'S1' THEN 20
    WHEN 'S2' THEN 50
    WHEN 'S3' THEN 100
    ELSE 50 -- assume Web/Business DBs
    END
    SELECT @db_size = SUM(reserved_page_count) * 8.0/1024/1024 FROM sys.dm_db_partition_stats
    SELECT @dbname as database_name, dateadd(second, round(datediff(second, '2015-01-01', end_time) / 15.0, 0) * 15,'2015-01-01')
    as end_time, avg_cpu_percent * (@DTU_cap/100.0) AS avg_cpu, avg_data_io_percent * (@DTU_cap/100.0) AS avg_io, avg_log_write_percent * (@DTU_cap/100.0) AS avg_log, @db_size as db_size FROM sys.dm_db_resource_stats
    WHERE end_time > '$($start_time)' and end_time <= '$($end_time)';
    " 
    }
    else
    {
    $sql = "Declare @dbname varchar(128) = '$($db.DatabaseName)';"
    $sql += "Declare @SLO varchar(20) = '$($db.CurrentServiceLevelObjectiveName)';"
    $sql+= "
    Declare @DTU_cap int, @db_size float;
    Select @DTU_cap = CASE @SLO 
    WHEN 'Basic' THEN 5
    WHEN 'S0' THEN 10
    WHEN 'S1' THEN 20
    WHEN 'S2' THEN 50
    WHEN 'P1' THEN 100 -- for old version of SQL database, P1 has same performance capacity as S3.
    ELSE 50 -- assume Web/Business DBs
    END
    SELECT @db_size = SUM(reserved_page_count) * 8.0/1024/1024 from sys.dm_db_partition_stats
    SELECT @dbname as database_name, dateadd(second, round(datediff(second, '2015-01-01', end_time) / 15.0, 0) * 15,'2015-01-01')
    as end_time, avg_cpu_percent * (@DTU_cap/100.0) AS avg_cpu, avg_data_io_percent * (@DTU_cap/100.0) AS avg_io, avg_log_write_percent * (@DTU_cap/100.0) AS avg_log, @db_size as db_size FROM sys.dm_db_resource_stats
    WHERE end_time > '$($start_time)' and end_time <= '$($end_time)';
    " 
    }
    #write-host $sql
    
    $result = Invoke-Sqlcmd -ServerInstance $servername -Database $db.DatabaseName -Username $username -Password $serverPassword -Query $sql -ConnectionTimeout 120 -QueryTimeout 3600 
    #bulk copy the metrics to output database
    $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $outputConnectionString 
    $bulkCopy.BulkCopyTimeout = 600
    $bulkCopy.DestinationTableName = "$destinationTableName";
    $bulkCopy.WriteToServer($result);
    
     }
    
     $start_time = $start_time.AddMinutes($Waittime)
     $end_time = $end_time.AddMinutes($Waittime)
     Write-Host $start_time
     Write-Host $end_time
     do {
    Start-Sleep 1
    }
    until (([DateTime]::UtcNow) -ge $end_time)
    }
    
    
    # Analysis query that does aggregation of the resource metrics to calculate pool size.
    $sql = 'Declare @DTU_Perf_99 as float, @DTU_Storage as float;
    WITH group_stats AS
    (
    SELECT end_time, SUM(db_size) AS avg_group_Storage, SUM(avg_cpu) AS avg_group_cpu, SUM(avg_io) AS avg_group_io,SUM(avg_log) AS avg_group_log
    FROM resource_stats_output
    GROUP BY end_time
    )
    -- calculate aggregate storage and DTUs for all DBs in the group
    , group_DTU AS
    (
    SELECT end_time, avg_group_Storage, 
    (SELECT Max(v)
       FROM (VALUES (avg_group_cpu), (avg_group_log), (avg_group_io)) AS value(v)) AS avg_group_DTU
    FROM group_stats
    )
    -- Get top 1 percent of the storage and DTU utilization samples.
    , top1_percent AS (
    SELECT TOP 1 PERCENT avg_group_Storage, avg_group_dtu FROM group_dtu ORDER BY [avg_group_DTU] DESC
    )
    
    -- Max and 99th percentile DTU for the given list of databases if converted into an elastic pool. Storage is increased by factor of 1.25 to accommodate for future growth. Currently storage limit of the pool is determined by the amount of DTUs based on 1GB/DTU.
    --SELECT MAX(avg_group_Storage)*1.25/1024.0 AS Group_Storage_DTU, MAX(avg_group_dtu) AS Group_Performance_DTU, MIN(avg_group_dtu) AS Group_Performance_DTU_99th_percentile FROM top1_percent;
    SELECT @DTU_Storage = MAX(avg_group_Storage)*1.25/1024.0, @DTU_Perf_99 = MIN(avg_group_dtu) FROM top1_percent;
    IF @DTU_Storage > @DTU_Perf_99 
    SELECT @DTU_Storage AS "Pool Size DTU dominated by storage"
    ELSE 
    SELECT @DTU_Perf_99 AS "Pool Size DTU dominated by resource consumption";'
    
    #write-host $sql
    Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $sql -QueryTimeout 3600
    
    

## Summary

Not all single databases are optimum candidates for elastic database pools. Databases with usage patterns that are characterized by low average utilization and relatively infrequent utilization spikes are excellent candidates for elastic pools. Application usage patterns are dynamic, so use the information and tools described in this article to make an initial assessment to see if an elastic database pool is a good choice for some or all of your databases. This article is just a starting point to help with your decision as to whether or not an elastic database pool is a good fit. Remember that you should continually monitor historical resource usage (using STA or DMVs) and constantly reassess the performance levels of all of your databases. Keep in mind that you can easily move databases in and out of elastic pools, and if you have a very large number of databases you can have multiple pools of varying sizes that you can divide your databases into.  



<!--Image references-->
[1]: ./media/sql-database-elastic-pool-guidance/one-database.png
[2]: ./media/sql-database-elastic-pool-guidance/four-databases.png
[3]: ./media/sql-database-elastic-pool-guidance/twenty-databases.png
