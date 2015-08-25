<properties
   pageTitle="Azure Data Catalog what is data catalog"
   description="Microsoft Azure Data Catalog is a fully managed cloud service that serves as a system of registration and system of discovery for enterprise data sources. Azure Data Catalog provides capabilities that enable any user – from analysts to data scientists to developers – to register, discover, understand, and consume data sources."
   services="data-catalog"
   documentationCenter=""
   authors="dvana"
   manager="mblythe"
   editor=""
   tags=""/>
<tags
   ms.service="data-catalog"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-catalog"
   ms.date="07/13/2015"
   ms.author="derrickv"/>

# What is Azure Data Catalog?

Microsoft **Azure Data Catalog** is a fully managed cloud service that serves as a system of registration and system of discovery for enterprise data sources. **Azure Data Catalog** provides capabilities that enable any user – from analysts to data scientists to developers – to register, discover, understand, and consume data sources.

## Problem description - motivation and overview

Traditionally, discovering enterprise data sources has been an organic process based on tribal knowledge. This presents numerous challenges for companies wanting to get the most value from their information assets.

-	Users are not aware that data sources exist unless they come into contact with it as part of another process; there is no central location where data sources are registered.
-	Unless a user knows the location of a data source, he cannot connect to the data using a client application; data consumption experiences require users to know the connection string or path.
-	Unless a user knows the location of a data source's documentation, he cannot understand the intended uses of the data; data sources and documentation live in different places and are consumed through different experiences.
-	If a user has questions about an information asset, he must locate the expert or team responsible for the data and engage those experts offline; there is no explicit connection between data and those with expert perspectives on its use.
 
While these challenges face data consumers, users responsible for producing and maintaining information assets face challenges of their own.

-	Annotating data sources with descriptive metadata is often a lost effort; client applications typically ignore descriptions stored in the data source. 
-	Creating documentation for data sources is often a lost effort; keeping documentation in sync with the data source is an ongoing responsibility, and users lack trust in documentation as it is often perceived as being out of date.

Creating and maintaining documentation for a data source is complex and time-consuming. The challenge of making that documentation readily available to everyone who uses the data source is often even more so.

When combined, these challenges present a significant barrier for companies who want to encourage and promote the use and understanding of enterprise data.

## Service description

**Azure Data Catalog** is designed to address these problems and to enable enterprises to get the most value from their existing information assets, by making them easily discoverable and understandable by the users who need the data they manage. 

**Azure Data Catalog** provides a cloud-based service into which data source can be registered. The data remains in in its existing location, but a copy of the metadata, along with a reference to the data source location, is added to **Azure Data Catalog**. This metadata is also indexed to make each data source easily discoverable via search.

Once a data source has been registered, its metadata can then be enriched, either by the user who performed the registration, or by other users in the enterprise. Any user can annotate a data source by providing descriptions, tags, or other metadata, such as documentation. This descriptive metadata supplements the structural metadata (such as column names and data types) registered from the data source, to make discovering and understanding it easier.

Discovering and understanding data sources and their use is the primary purpose of registering the sources. When enterprise users need data for their efforts (which could be business intelligence, application development, data science, or any other task where the right data is required) they can use the **Azure Data Catalog** discovery experience to quickly find data that matches their needs, understand the data to evaluate its fitness for purpose, and consume that data by opening the data source in their tool of choice.

## Register data sources

Data source registration is performed using the **Azure Data Catalog** data source registration tool. This   application can be downloaded from the **Azure Data Catalog** portal.

The registration process involves three basic steps:
 
1.	Connect to a data source - the user specifies the data source location and the credentials to connect to the data source, such as a SQL Server instance.
2.	Select objects to register - the user selects the objects in the specified location that should be registered with **Azure Data Catalog**. This may be the full set of tables in all databases on the server, or a specifically selected subset of tables and views.
3.	Complete registration - the user completes the process, and the data source registration tool extracts the structural metadata from the data source, and sends that metadata to the **Azure Data Catalog** cloud service.
	
> [AZURE.NOTE] For preview, **Azure Data Catalog** currently supports the following data source and asset types:
 
- SQL Server Table
- SQL Server View
- Oracle Database Table
- Oracle Database View
- SQL Server Analysis Services Multidimensional Dimension
- SQL Server Analysis Services Multidimensional Measure
- SQL Server Analysis Services Multidimensional KPI 
- SQL Server Analysis Services Tabular Table
- SQL Server Reporting Services Report

Additional data sources and asset types will be added during the **Azure Data Catalog** preview.

> [AZURE.IMPORTANT]Registering a data source in **Azure Data Catalog** does not copy the data from the data source, unless you select “Include Preview” in the data source registration tool. Registration copies data source metadata, not data. Examples of metadata include the names of tables and other data source objects, along with the names and data types of columns and other data source attributes. The metadata also includes the data source location, so that users who discover the data source using **Azure Data Catalog** can then connect to the data source. If you select “Include Preview” then the data source registration tool will also copy to **Azure Data Catalog** a small set of records that will be displayed to users who discover the data source in the **Azure Data Catalog** portal.

## Enrich data source metadata

Once the registration is complete, the data sources can be discovered and consumed, but the true value of **Azure Data Catalog** comes from having descriptive business metadata in the same experience as the structural metadata extracted from the data source.  This additional metadata delivers two significant benefits:
 
-	The registered data sources are more easily discoverable. The user-provided metadata is added to the **Azure Data Catalog** search index. This allows users to discover the data by using terms and concepts that may not be present in the original data source. For example, if a database table that contains customer data is named "tbl_c45", providing a friendly name of "Customer" will make it more easily discoverable by users looking for customer data. Similarly, providing a description that includes the names of reports, dashboards, or processes that use the data will make the data source easier to find by users who use those downstream artifacts as their search terms.
-	The registered data sources are more easily understood once discovered. The user-provided metadata is presented to any **Azure Data Catalog** user who views the annotated data source, which helps provide additional context and information. Most data sources typically do not include meaningful descriptions or documentation, and those that do are often focused on the technical DBA or database developer audiences. By enriching data sources in **Azure Data Catalog** with audience-appropriate descriptions and tags, users can help ensure that those who discover the data can understand its details and intended use.
  
> [AZURE.NOTE] Each **Azure Data Catalog** user can add his own tags and descriptions for data assets and columns. **Azure Data Catalog** will track the value and source of each annotation and will display the user and the date the annotation was added. This crowdsourcing approach to metadata ensures that every user with a perspective on the data and its use can share their opinions and resources with the user community at large.

## Explore, discover, and understand

The goal of registering and enriching data sources in **Azure Data Catalog** is so that they can be discovered, understood, and used by users across the enterprise. The **Azure Data Catalog** portal is the primary tool for this process.

The **Azure Data Catalog** portal provides two primary mechanisms for data exploration and discovery: searching and filtering. 

To search **Azure Data Catalog** for data sources, simply enter a search term in the search box in the **Azure Data Catalog** portal. The portal will display a tile for each registered data source that matches the search term; the tiles will contain the name, description, and tags assigned to the data source, along with other high-level information.

To filter the contents of **Azure Data Catalog**, simply select one or more of the filters presented in the **Azure Data Catalog** portal. This will restrict the tiles displayed in the portal to only those matching the specified filter criteria. You can filter the data sources without searching, or you can filter the results of a search.

To view more complete information for a data source, and to understand if it is appropriate for the task at hand, simply click on the tile for the data source; the properties pane will be displayed, and will contain all of its metadata. 

At the top of the properties pane there will be additional buttons:
 
1.	Preview: Selecting this button will display the static set of preview records from the data source, if preview was selected during data source registration.
2.	Schema: Selecting this button will display the schema for the data source, including column names and data types, and any column-level metadata in **Azure Data Catalog**.

> [AZURE.NOTE] It's important to remember that the **Discover** experience can be an entry point into the **Enrich** experience, and not only into the **Consume** experience. The crowdsourcing approach that **Azure Data Catalog** brings means that any user who discovers a registered data source can share his opinions on the data, in addition to using the data he has discovered.

## Remove data source metadata

After a data source has been registered, it can sometimes be necessary to remove the data source reference from **Azure Data Catalog**. This may be due to changing business requirements, or to the source system being retired. Regardless of the reason, **Azure Data Catalog** makes it easy to remove data sources by simply selecting to delete so that they can no longer be discovered and consumed.

> [AZURE.IMPORTANT] Deleting a data source from **Azure Data Catalog** only deletes the metadata stored in the **Azure Data Catalog** service. The original data source is not affected in any way.

## Consume data sources

The ultimate goal of data discovery is to find the data that you need, and to use it in the data tool of your choice. The data consumption experience in Azure Data Catalog enables this capability in two ways.
 
1.	For client applications that are directly supported by **Azure Data Catalog**, users can click on the **Connect** menu in the data source tile found in the portal. The client application will then launch with a connection to the selected data source.
2.	For all client applications, users can use the connection information displayed in the properties pane for a selected data source. This information includes all details (such as server name, database name, and object name) required to connect to the data, and can be copied into the client tool's connection experience.
 
> [AZURE.NOTE] For the private preview of Azure Data Catalog, only Microsoft Excel will be directly supported and available in the **Connect** menu.
