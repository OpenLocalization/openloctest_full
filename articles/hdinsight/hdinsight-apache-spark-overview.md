<properties 
	pageTitle="An overview of Apache Spark in HDInsight | Azure" 
	description="An introduction to Apache Spark in HDInsight and scenarios in which to use Spark on HDInsight in your applications." 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="hdinsight" 
	ms.workload="big-data" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="07/10/2015" 
	ms.author="nitinme"/>

# Overview: Apache Spark on Azure HDInsight 
 
<a href="http://spark.apache.org/" target="_blank">Apache Spark</a> is an open-source parallel processing framework that supports in-memory processing to boost the performance of big-data analytic applications. Spark processing engine is built for speed, ease of use, and sophisticated analytics. Spark's in-memory computation capabilities make it a good choice for iterative algorithms in machine learning and graph computations. Spark is also compatible with Azure Blob storage (WASB) so your existing data stored in Azure can easily be processed via Spark.

When you provision a Spark cluster in HDInsight, you provision Azure compute resources with Spark installed and configured. It only takes about ten minutes to provision a Spark cluster in HDInsight. The data to be processed is stored in Azure Blob storage. See [Use Azure Blob Storage with HDInsight][hdinsight-storage].

![Apache Spark on Azure HDInsight](./media/hdinsight-apache-spark-overview/SparkArchitecture.png  "Apache Spark on Azure HDInsight")


**Want to get started with Apache Spark on Azure HDInsight?** See [QuickStart: Provision a Spark cluster on HDInsight and run sample applications using Jupyter and Zeppelin](hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql.md).



Watch a quick overview video describing Apache Spark on Azure HDInsight.

> [AZURE.VIDEO announcing-apache-spark-on-azure-hdinsight]

## Why use Spark on Azure HDInsight? 

Azure HDInsight offers a fully managed Spark service. Benefits of using Spark on HDInsight are:

| Feature                             | Description       |
|-------------------------------------|-------------------|
| Ease of provisioning            | You can provision a new Spark cluster on HDInsight in minutes using the Azure Management Portal, Azure PowerShell, or the HDInsight .NET SDK. See [Provision a Spark cluster in HDInsight](hdinsight-apache-spark-provision-clusters.md) |
| Ease of use                     | Spark in HDInsight clusters includes Zeppelin and Jupyter notebooks pre-configured. You can use these for interactive data processing and visualization. You can launch these notebooks from the cluster dashboard to work directly against a Spark cluster.|
| REST APIs                       | Spark in HDInsight includes Spark job server, which is a REST API server that enables users to remotely submit and monitor running jobs. |
| Concurrent Queries              | Spark in HDInsight supports concurrent queries. This enables multiple queries from one user or multiple queries from various users and applications to share the same cluster resources. |
| Caching on SSDs                 | You can choose to cache data either in memory or in SSDs attached to the cluster nodes. Caching in memory provides the best query performance but could be expensive; caching in SSDs provides a great option for improving query performance without the need to create a cluster of a size that is required to fit the entire dataset in memory.|
| Integration with Azure services | Spark on HDInsight comes with a connector to Azure Event Hubs. Customers can build streaming applications using the Event Hubs, in addition to [Kafka](http://kafka.apache.org/), which is already available as part of Spark. |
| Integration with BI Tools       | Spark for HDInsight provides connectors for popular BI tools such as [Power BI](http://www.powerbi.com/) and [Tableau](http://www.tableau.com/products/desktop) for data analytics.|
| Pre-loaded Anaconda libraries        | Spark clusters on HDInsight come with Anaconda libraries pre-installed. [Anaconda](http://docs.continuum.io/anaconda/) provides close to 200 libraries for machine learning, data analysis, visualization, etc.|
| Scalability                     | Although you can specify the number of nodes in your cluster during creation, you may want to grow or shrink the cluster to match workload. All HDInsight clusters allow you to change the number of nodes in the cluster. Also, Spark clusters can be dropped with no loss of data since all the data is stored in Azure Blob Storage. |
| 24/7 Support					  | Spark on HDInsight comes with  enterprise-level 24/7 support and an SLA of 99.9% up-time.|



## What are the use cases for Spark on HDInsight?

Apache Spark in HDInsight enables the following key scenarios.

### Interactive data analysis and BI

[Look at a tutorial](hdinsight-apache-spark-use-bi-tools.md)

Apache Spark in HDInsight stores data in Azure Blobs. Business experts and key decision makers can analyze and build reports over that data and use Microsoft Power BI to build interactive reports from the analyzed data. Analysts can start from unstructured/semi structured data in Azure storage, define a schema for the data using notebooks and then build data models using Microsoft Power BI. Spark in HDInsight also supports a number of third party BI tools such as Tableau, Qlikview, and SAP Lumira making it an ideal platform for data analysts, business experts, and key decision makers.

### Iterative Machine Learning

[Look at a tutorial](hdinsight-apache-spark-ipython-notebook-machine-learning.md)

Apache Spark comes with [MLlib](http://spark.apache.org/mllib/), a machine learning library built on top of Spark. In addition to this, Spark on HDInsight also includes Anaconda, a Python distribution with a variety of packages for machine learning. Couple this with a built-in support for IPython notebooks, and you have a top-of-the-line environment for creating machine learning applications.  

### Streaming and real-time data analysis

[Look at a tutorial](hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming.md)

Real-time data analysis is used for scenarios ranging from reducing time to data insight by processing data as it lands, to building a true streaming solution. Spark in HDInsight offers a rich support for building real-time analytics solutions. While Spark already has connectors to ingest data from many sources like Kafka, Flume, Twitter, ZeroMQ, or TCP sockets, Spark in HDInsight adds first-class support for ingesting data from Azure Event Hubs. Event Hubs are the most widely used queuing service on Azure. Having an out-of-the-box support for Event Hubs makes Spark in HDInsight an ideal platform for building real time analytics pipeline.

##<a name="next-steps"></a>What components are included as part of a Spark cluster?

Spark in HDInsight includes the following components that are available on the clusters by default.

- [Spark 1.3.1](https://spark.apache.org/docs/1.3.1/). Comes with Spark Core, Spark SQL, Spark streaming APIs, GraphX, and MLlib.
- [Anaconda](http://docs.continuum.io/anaconda/)
- [Spark Job Server](https://github.com/spark-jobserver/spark-jobserver)
- [Zeppelin Notebook](https://zeppelin.incubator.apache.org)
- [IPython Notebook](https://jupyter.org)

Spark in HDInsight also provides an [ODBC driver](http://go.microsoft.com/fwlink/?LinkId=616229) for connectivity to Spark clusters in HDInsight from BI tools such as Microsoft Power BI and Tableau.

##<a name="see-also"></a>See also

* [QuickStart: Use Spark in HDInsight with Zeppelin Notebook to perform interactive data analysis](hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql.md)
* [Provision an Apache Spark cluster in HDInsight](hdinsight-apache-spark-provision-clusters.md)
* [Perform interactive data analysis using Spark in HDInsight with BI tools](hdinsight-apache-spark-use-bi-tools.md)
* [Use Spark in HDInsight for building machine learning applications](hdinsight-apache-spark-ipython-notebook-machine-learning.md)
* [Use Spark in HDInsight for building real-time streaming applications](hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming.md)
* [Manage resources for the Apache Spark cluster in Azure HDInsight](hdinsight-apache-spark-resource-manager.md)
* [Submit jobs remotely to an Apache Spark cluster on Azure HDInsight](hdinsight-apache-spark-job-server.md)


[hdinsight-storage]: ../hdinsight-use-blob-storage/