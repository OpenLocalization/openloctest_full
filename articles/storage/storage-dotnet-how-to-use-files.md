<properties
			pageTitle="How to use Azure File storage with PowerShell and .NET | Microsoft Azure"
            description="Learn how to use Azure File storage to create cloud file shares and manage file content. File storage enables enterprises to move applications that rely on SMB file shares to Azure. Persist your storage account credentials for the virtual machine so as to reconnect to the file share on reboot."
            services="storage"
            documentationCenter=".net"
            authors="tamram"
            manager="adinah"
            editor="" />

<tags ms.service="storage"
      ms.workload="storage"
      ms.tgt_pltfrm="na"
      ms.devlang="dotnet"
      ms.topic="hero-article"
      ms.date="07/06/2015"
      ms.author="tamram" />

# How to use Azure File storage with PowerShell and .NET

## Overview

The Azure File service exposes file shares using the standard SMB 2.1 protocol. Applications running in Azure can now easily share files between VMs using standard and familiar file system APIs like ReadFile and WriteFile. In addition, the files can also be accessed at the same time via a REST interface, which opens a variety of hybrid scenarios. Finally, Azure Files is built on the same technology as the Blob, Table, and Queue Services, which means Azure Files is able to leverage the existing availability, durability, scalability, and geo redundancy that is built into our platform.

## About this tutorial

This getting started tutorial demonstrates the basics of using Microsoft Azure File storage. In this tutorial, we will:

- Use PowerShell to show how to create a new Azure File share, add a directory, upload a local file to the share, and list the files in the directory.
- Mount the file share from an Azure virtual machine, just as you would any SMB share.

For users who may want to access files in a share from an on-premises application as well as from an Azure virtual machine or cloud service, we show how to use the Azure .NET Storage Client Library to work with the file share from a desktop application.

> [AZURE.NOTE] Running the .NET code examples in this guide requires the Azure .NET Storage Client Library 4.x or later. The Storage Client Library is available via [NuGet](https://www.nuget.org/packages/WindowsAzure.Storage/).

[AZURE.INCLUDE [storage-file-concepts-include](../../includes/storage-file-concepts-include.md)]


## Create an Azure Storage account

Azure File storage is currently in preview. To request access to the preview, navigate to the [Microsoft Azure Preview page](/services/preview/), and request access to **Azure Files**. Once your request is approved, you'll be notified that you can access the File storage preview. You can then create a storage account for accessing File storage.

> [AZURE.NOTE] File storage is currently available only for new storage accounts. After your subscription is granted access to File storage, create a new storage account for use with this guide.
>
> Azure File storage does not currently support shared access signatures.

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## Use PowerShell to create a file share

Next, we'll use Azure PowerShell to create a file share. Once the file share has been created, you can mount it from any file system that supports SMB 2.1.

### Install the PowerShell cmdlets for Azure Storage

To prepare to use PowerShell, download and install the Azure PowerShell cmdlets. See [How to install and configure Azure PowerShell](../install-configure-powershell.md) for the install point and installation instructions.

> [AZURE.NOTE] The PowerShell cmdlets for the File service are available only in the latest Azure PowerShell module, version 0.8.5 and later. It's recommended that you download and install or upgrade to the latest Azure PowerShell module.

Open an Azure PowerShell window by clicking **Start** and typing **Azure PowerShell**. The Azure PowerShell window loads the Azure Powershell module for you.

### Create a context for your storage account and key

Now, create the storage account context. The context encapsulates the storage account name and account key. For instructions on copying your account key from the Azure portal, see [View, copy, and regenerate storage access keys](storage-create-storage-account.md#view-copy-and-regenerate-storage-access-keys).

Replace `storage-account-name` and `storage-account-key` with your storage account name and key in the following example:

	# create a context for account and key
	$ctx=New-AzureStorageContext storage-account-name storage-account-key

### Create a new file share

Next, create the new share, named `logs` in this example:

	# create a new share
	$s = New-AzureStorageShare logs -Context $ctx

You now have a file share in File storage. Next we'll add a directory and a file.

> [AZURE.IMPORTANT] The name of your file share must be all lowercase. For complete details on naming file shares and files, see [Naming and Referencing Shares, Directories, Files, and Metadata](https://msdn.microsoft.com/library/azure/dn167011.aspx).

### Create a directory in the file share

Next, create a directory in the share. In the following example, the directory is named `CustomLogs`:

    # create a directory in the share
    New-AzureStorageDirectory -Share $s -Path CustomLogs

### Upload a local file to the directory

Now upload a local file to the directory. The following example uploads a file from `C:\temp\Log1.txt`. Edit the file path so that it points to a valid file on your local machine:

    # upload a local file to the new directory
    Set-AzureStorageFileContent -Share $s -Source C:\temp\Log1.txt -Path CustomLogs

### List the files in the directory

To see the file in the directory, you can list the directory's files. This command will also list subdirectories, but in this example, there is no subdirectory, so only the file will be listed.

	# list files in the new directory
	Get-AzureStorageFile -Share $s -Path CustomLogs

## Mount the share from an Azure virtual machine running Windows

To demonstrate how to mount an Azure file share, we'll now create an Azure virtual machine running Windows, and remote into it to mount the share.

1. First, create a new Azure virtual machine by following the instructions in [Create a Virtual Machine Running Windows Server](../virtual-machines-windows-tutorial.md).
2. Next, remote into the virtual machine by following the instructions in [How to Log on to a Virtual Machine Running Windows Server](../virtual-machines-log-on-windows-server.md).
3. Open a PowerShell window on the virtual machine.

### Persist your storage account credentials for the virtual machine

Before mounting to the file share, first persist your storage account credentials on the virtual machine. This step allows Windows to automatically reconnect to the file share when the virtual machine reboots. To persist your account credentials, execute the `cmdkey` command from the PowerShell window on the virtual machine. Replace `<storage-account-name>` with the name of your storage account, and `<storage-account-key>` with your storage account key:

	cmdkey /add:<storage-account-name>.file.core.windows.net /user:<storage-account-name> /pass:<storage-account-key>

Windows will now reconnect to your file share when the virtual machine reboots. You can verify that the share has been reconnected by executing the `net use` command from a PowerShell window.

### Mount the file share using the persisted credentials

Once you have a remote connection to the virtual machine, you can execute the `net use` command to mount the file share, using the following syntax. Replace `<storage-account-name>` with the name of your storage account, and `<share-name>` with the name of your File storage share.

    net use <drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name>

	example :
	net use z: \\samples.file.core.windows.net\logs

> [AZURE.NOTE] Since you persisted your storage account credentials in the previous step, you do not need to provide them with the `net use` command. If you have not already persisted your credentials, then include them as a parameter passed to the `net use` command.

    net use <drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> /u:<storage-account-name> <storage-account-key>

	example :
	net use z: \\samples.file.core.windows.net\logs /u:samples <storage-account-key>

You can now work with the File Storage share from the virtual machine as you
would with any other drive. You can issue standard file commands from the command prompt, or view the mounted share and its contents from File Explorer. You can also run code within the virtual machine that accesses the file share using standard Windows file I/O APIs, such as those provided by the [System.IO namespaces](http://msdn.microsoft.com/library/gg145019(v=vs.110).aspx) in the .NET Framework.

You can also mount the file share from a role running in an Azure cloud service by remoting into the role.

## Create an on-premises application to work with File storage

You can mount the file share from a virtual machine or a cloud service running in Azure, as demonstrated above. However, you cannot mount the file share from an on-premises application. To access share data from an on-premises application, you must use the File storage API. This example demonstrates how to work with a file share via the [Azure .NET Storage Client Library](http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409).

To show how to use the API from an on-premises application, we'll create a simple console application running on the desktop.

### Create the console application and obtain the assembly

To create a new console application in Visual Studio and install the Azure Storage NuGet package:

1. In Visual Studio, choose **File** -> **New Project**, and choose **Windows** -> **Console Application** from the list of Visual C# templates.
2. Provide a name for the console application, and click **OK**.
3. Once your project has been created, right-click the project in Solution Explorer and choose **Manage NuGet Packages**. Search online for "WindowsAzure.Storage" and click **Install** to install the Azure Storage package and dependencies.

### Save your storage account credentials to the app.config file

Next, save your credentials in your project's app.config file. Edit the app.config file so that it appears similar to the following example, replacing `myaccount` with your storage account name, and `mykey` with your storage account key:

	<?xml version="1.0" encoding="utf-8" ?>
	<configuration>
	    <startup>
	        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
	    </startup>
	    <appSettings>
	        <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=StorageAccountKeyEndingIn==" />
	    </appSettings>
	</configuration>

> [AZURE.NOTE] The latest version of the Azure storage emulator does not support File storage. Your connection string must target an Azure storage account in the cloud with access to the Files preview.


### Add namespace declarations

Open the program.cs file from Solution Explorer, and add the following namespace declarations to the top of the file:

	using Microsoft.WindowsAzure;
	using Microsoft.WindowsAzure.Storage;
	using Microsoft.WindowsAzure.Storage.File;

### Retrieve your connection string programmatically

You can retrieve your saved credentials from the app.config file using either the `Microsoft.WindowsAzure.CloudConfigurationManager` class, or the `System.Configuration.ConfigurationManager `class. The example here shows how to retrieve your credentials using the `CloudConfigurationManager` class and encapsulate them with the `CloudStorageAccount` class. Add the following code to the `Main()` method in program.cs:

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    	CloudConfigurationManager.GetSetting("StorageConnectionString"));

### Access the File storage share programmatically

Next, add the following code to the `Main()` method, after the code shown above to retrieve the connection string. This code gets a reference to the file we created earlier and outputs its contents to the console window.

	//Create a CloudFileClient object for credentialed access to File storage.
	CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

	//Get a reference to the file share we created previously.
	CloudFileShare share = fileClient.GetShareReference("logs");

	//Ensure that the share exists.
	if (share.Exists())
	{
	    //Get a reference to the root directory for the share.
	    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

	    //Get a reference to the directory we created previously.
	    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

	    //Ensure that the directory exists.
	    if (sampleDir.Exists())
	    {
	        //Get a reference to the file we created previously.
	        CloudFile file = sampleDir.GetFileReference("Log1.txt");

	        //Ensure that the file exists.
	        if (file.Exists())
	        {
	            //Write the contents of the file to the console window.
	            Console.WriteLine(file.DownloadTextAsync().Result);
	        }
	    }
	}

Run the console application to see the output.

## Mount the share from an Azure virtual machine running Linux

When you create an Azure virtual machine, you can specify an Ubuntu image from the disk images gallery to ensure support for SMB 2.1. However, any Linux distribution that supports SMB 2.1 can mount an Azure File share.

For a demonstration of how to mount an Azure File share on Linux, see [Shared storage on Linux via Azure Files Preview - Part 1](http://channel9.msdn.com/Blogs/Open/Shared-storage-on-Linux-via-Azure-Files-Preview-Part-1) on Channel 9.

## Next steps

See these links for more information about Azure File storage.

### Reference

- [Storage Client Library for .NET reference](https://msdn.microsoft.com/library/azure/dn261237.aspx)
- [File Service REST API reference](http://msdn.microsoft.com/library/azure/dn167006.aspx)

### Blog posts

- [Introducing Microsoft Azure File Service](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)
- [Persisting connections to Microsoft Azure Files](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)
 