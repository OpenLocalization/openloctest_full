<properties 
	pageTitle="Create a NoSQL DocumentDB database | Microsoft Azure" 
	description="Learn how to create managed databases using the online service portal for Azure DocumentDB, a NoSQL document database for JSON. Get a free trial today." 
	services="documentdb" 
	authors="mimig1" 
	manager="jhubbard" 
	editor="monicar" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="get-started-article" 
	ms.date="06/26/2015" 
	ms.author="mimig"/>

# Create a DocumentDB database using the Azure preview portal

To use Microsoft Azure DocumentDB, you must have a [DocumentDB account](documentdb-create-account.md), a database, a collection, and documents.  This topic describes how to create a DocumentDB database in the Microsoft Azure preview portal. 

Databases do not have to be created using the preview portal, you can also create them using the [DocumentDB SDKs](https://msdn.microsoft.com/library/azure/dn781482.aspx). For a C# code sample showing how to create a database using the DocumentDB .NET SDK, see the [Program.cs](https://github.com/Azure/azure-documentdb-net/blob/master/samples/code-samples/DatabaseManagement/Program.cs) file in the DatabaseManagement project, available in the [azure-documentdb-net](https://github.com/Azure/azure-documentdb-net) repository on [GitHub.com](https://github.com). 

1.  In the [Azure preview portal](https://portal.azure.com/), click **Browse all**.

2.  In the **Browse** blade, click **DocumentDB Accounts**.

3.  In the **DocumentDB Accounts** blade, select the account in which to add a DocumentDB database. If you don't have any accounts listed, you'll need to [create a DocumentDB account](documentdb-create-account.md).
    
    ![Screen shot highlighting the Browse button, DocumentDB Accounts on the Browse blade, and a DocumentDB account on the DocumentDB Accounts blade](./media/documentdb-create-database/docdb-database-creation-1-3.png)

4. In the **DocumentDB Account** blade, click **Add Database**.

5. In the **Add Database** blade, enter the ID for your new database. When the name is validated, a green check mark appears in the ID box.

6. Click **OK** at the bottom of the screen to create the new database. 

	![Screen shot highlighting the Add Database button on the DocumentDB Account blade, the ID box on the Add Database blade, and the OK button](./media/documentdb-create-database/docdb-database-creation-4-6.png)

8. The new database now appears in the **Databases** lens on the **DocumentDB Account** blade.
 
	![Screen shot of the new database in the DocumentDB Account blade](./media/documentdb-create-database/docdb-database-creation-7.png)

## Next steps

Now that you have a DocumentDB database, the next step is to [create a collection](documentdb-create-collection.md).

Once your collection is created, you can [add documents](../documentdb-view-json-document-explorer.md) by using the Document Explorer in the preview portal, [import documents](documentdb-import-data.md) into the collection by using the DocumentDB Data Migration Tool, or use one of the [DocumentDB SDKs](https://msdn.microsoft.com/library/azure/dn781482.aspx) to perform CRUD operations. DocumentDB has .NET, Java, Python, Node.js, and JavaScript API SDKs. For .NET code samples showing how to create, remove, update and delete documents, see [Program.cs](https://github.com/Azure/azure-documentdb-net/blob/master/samples/code-samples/DocumentManagement/Program.cs) in the DocumentManagement project in the azure-documentdb-net repository on GitHub.com.  

After you have documents in a collection, you can use [DocumentDB SQL](documentdb-sql-query.md) to [execute queries](documentdb-sql-query.md#executing-queries) against your documents by using the [Query Explorer](documentdb-query-collections-query-explorer.md) in the preview portal, the [REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx), or one of the [SDKs](https://msdn.microsoft.com/library/azure/dn781482.aspx). 