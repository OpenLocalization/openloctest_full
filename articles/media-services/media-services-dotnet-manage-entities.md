
<properties 
	pageTitle="Managing Assets and Related Entities with Media Services .NET SDK" 
	description="Learn how to manage assets and related entities with the Media Services SDK for .NET." 
	authors="juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/29/2015" 
	ms.author="juliako"/>


#Managing Assets and Related Entities with Media Services .NET SDK

This topic shows how to accomplish the following Media Services management tasks:

- Get an asset reference 
- Get a job reference 
- List all assets 
- List jobs and assets 
- List all access policies 
- List All Locators 
- Delete an asset 
- Delete a job 
- Delete an access policy 

##Prerequisites 

See [Set up your environment](media-services-set-up-computer.md)

##Get an Asset Reference

A frequent task is to get a reference to an existing asset in Media Services. The following code example shows how you can get an asset reference from the Assets collection on the server context object, based on an asset Id.
The following code example uses a Linq query to get a reference to an existing IAsset object.

	static IAsset GetAsset(string assetId)
	{
	    // Use a LINQ Select query to get an asset.
	    var assetInstance =
	        from a in _context.Assets
	        where a.Id == assetId
	        select a;
	    // Reference the asset as an IAsset.
	    IAsset asset = assetInstance.FirstOrDefault();
	
	    return asset;
	}

##Get a Job Reference

When you work with processing tasks in Media Services code, you often need to get a reference to an existing job based on an Id. The following code example shows how to get a reference to an IJob object from the Jobs collection.
WarningWarning
You may need to get a job reference when starting a long-running encoding job, and need to check the job status on a thread. In cases like this, when the method returns from a thread, you need to retrieve a refreshed reference to a job.

	static IJob GetJob(string jobId)
	{
	    // Use a Linq select query to get an updated 
	    // reference by Id. 
	    var jobInstance =
	        from j in _context.Jobs
	        where j.Id == jobId
	        select j;
	    // Return the job reference as an Ijob. 
	    IJob job = jobInstance.FirstOrDefault();
	
	    return job;
	}

##List All Assets

As the number of assets you have in storage grows, it is helpful to list your assets. The following code example shows how to iterate through the Assets collection on the server context object. With each asset, the code example also writes some of its property values to the console. For example, each asset can contain many media files. The code example writes out all files associated with each asset.



	static void ListAssets()
	{
	    string waitMessage = "Building the list. This may take a few "
	        + "seconds to a few minutes depending on how many assets "
	        + "you have."
	        + Environment.NewLine + Environment.NewLine
	        + "Please wait..."
	        + Environment.NewLine;
	    Console.Write(waitMessage);
	
	    // Create a Stringbuilder to store the list that we build. 
	    StringBuilder builder = new StringBuilder();
	
	    foreach (IAsset asset in _context.Assets)
	    {
	        // Display the collection of assets.
	        builder.AppendLine("");
	        builder.AppendLine("******ASSET******");
	        builder.AppendLine("Asset ID: " + asset.Id);
	        builder.AppendLine("Name: " + asset.Name);
	        builder.AppendLine("==============");
	        builder.AppendLine("******ASSET FILES******");
	
	        // Display the files associated with each asset. 
	        foreach (IAssetFile fileItem in asset.AssetFiles)
	        {
	            builder.AppendLine("Name: " + fileItem.Name);
	            builder.AppendLine("Size: " + fileItem.ContentFileSize);
	            builder.AppendLine("==============");
	        }
	    }
	
	    // Display output in console.
	    Console.Write(builder.ToString());
	}

##List Jobs and Assets

An important related task is to list assets with their associated job in Media Services. The following code example shows you how to list each IJob object, and then for each job, it displays properties about the job, all related tasks, all input assets, and all output assets. The code in this example can be useful for numerous other tasks. For example, if you want to list the output assets from one or more encoding jobs that you ran previously, this code shows how to access the output assets. When you have a reference to an output asset, you can then deliver the content to other users or applications by downloading it, or providing URLs. 

For more information on options for delivering assets, see [Deliver Assets with the Media Services SDK for .NET](media-services-deliver-streaming-content.md).

	// List all jobs on the server, and for each job, also list 
	// all tasks, all input assets, all output assets.

	static void ListJobsAndAssets()
	{
	    string waitMessage = "Building the list. This may take a few "
	        + "seconds to a few minutes depending on how many assets "
	        + "you have."
	        + Environment.NewLine + Environment.NewLine
	        + "Please wait..."
	        + Environment.NewLine;
	    Console.Write(waitMessage);
	
	    // Create a Stringbuilder to store the list that we build. 
	    StringBuilder builder = new StringBuilder();
	
	    foreach (IJob job in _context.Jobs)
	    {
	        // Display the collection of jobs on the server.
	        builder.AppendLine("");
	        builder.AppendLine("******JOB*******");
	        builder.AppendLine("Job ID: " + job.Id);
	        builder.AppendLine("Name: " + job.Name);
	        builder.AppendLine("State: " + job.State);
	        builder.AppendLine("Order: " + job.Priority);
	        builder.AppendLine("==============");
	
	
	        // For each job, display the associated tasks (a job  
	        // has one or more tasks). 
	        builder.AppendLine("******TASKS*******");
	        foreach (ITask task in job.Tasks)
	        {
	            builder.AppendLine("Task Id: " + task.Id);
	            builder.AppendLine("Name: " + task.Name);
	            builder.AppendLine("Progress: " + task.Progress);
	            builder.AppendLine("Configuration: " + task.Configuration);
	            if (task.ErrorDetails != null)
	            {
	                builder.AppendLine("Error: " + task.ErrorDetails);
	            }
	            builder.AppendLine("==============");
	        }
	
	        // For each job, display the list of input media assets.
	        builder.AppendLine("******JOB INPUT MEDIA ASSETS*******");
	        foreach (IAsset inputAsset in job.InputMediaAssets)
	        {
	
	            if (inputAsset != null)
	            {
	                builder.AppendLine("Input Asset Id: " + inputAsset.Id);
	                builder.AppendLine("Name: " + inputAsset.Name);
	                builder.AppendLine("==============");
	            }
	        }
	
	        // For each job, display the list of output media assets.
	        builder.AppendLine("******JOB OUTPUT MEDIA ASSETS*******");
	        foreach (IAsset theAsset in job.OutputMediaAssets)
	        {
	            if (theAsset != null)
	            {
	                builder.AppendLine("Output Asset Id: " + theAsset.Id);
	                builder.AppendLine("Name: " + theAsset.Name);
	                builder.AppendLine("==============");
	            }
	        }
	
	    }
	
	    // Display output in console.
	    Console.Write(builder.ToString());
	}

##List All Access Policies

In Media Services, you can define an access policy on an asset or its files. An access policy defines the permissions for a file or an asset (what type of access, and the duration). In your Media Services code, you typically define an access policy by creating an IAccessPolicy object and then associating it with an existing asset. Then you create a ILocator object, which lets you provide direct access to assets in Media Services. The Visual Studio project that accompanies this documentation series contains several code examples that show how to create and assign access policies and locators to assets.

The following code example shows how to list all access policies on the server, and shows the type of permissions associated with each. Another useful way to view access policies is to list all ILocator objects on the server, and then for each locator, you can list its associated access policy by using its AccessPolicy property.

	static void ListAllPolicies()
	{
	    foreach (IAccessPolicy policy in _context.AccessPolicies)
	    {
	        Console.WriteLine("");
	        Console.WriteLine("Name:  " + policy.Name);
	        Console.WriteLine("ID:  " + policy.Id);
	        Console.WriteLine("Permissions: " + policy.Permissions);
	        Console.WriteLine("==============");
	
	    }
	}

##List All Locators

A locator is a URL that provides a direct path to access an asset, along with permissions to the asset as defined by the locator's associated access policy. Each asset can have a collection of ILocator objects associated with it on its Locators property. The server context also has a Locators collection that contains all locators.

The following code example lists all locators on the server. For each locator, it shows the Id for the related asset and access policy. It also displays the type of permissions, the expiration date, and the full path to the asset.

Note that a locator path to an asset is only a base URL to the asset. To create a direct path to individual files that a user or application could browse to, your code must add the specific file path to the locator path. For more information on how to do this, see the topic [Deliver Assets with the Media Services SDK for .NET](media-services-deliver-streaming-content.md).

	static void ListAllLocators()
	{
	    foreach (ILocator locator in _context.Locators)
	    {
	        Console.WriteLine("***********");
	        Console.WriteLine("Locator Id: " + locator.Id);
	        Console.WriteLine("Locator asset Id: " + locator.AssetId);
	        Console.WriteLine("Locator access policy Id: " + locator.AccessPolicyId);
	        Console.WriteLine("Access policy permissions: " + locator.AccessPolicy.Permissions);
	        Console.WriteLine("Locator expiration: " + locator.ExpirationDateTime);
	        // The locator path is the base or parent path (with included permissions) to access  
	        // the media content of an asset. To create a full URL to a specific media file, take 
	        // the locator path and then append a file name and info as needed.  
	        Console.WriteLine("Locator base path: " + locator.Path);
	        Console.WriteLine("");
	    }
	}


##Delete an Asset

The following example deletes an asset.

	static void DeleteAsset( IAsset asset)
	{
	    // delete the asset
	    asset.Delete();
	
	    // Verify asset deletion
	    if (GetAsset(asset.Id) == null)
	        Console.WriteLine("Deleted the Asset");
	
	}

##Delete a Job

To delete a job, you must check the state of the job as indicated in the State property. Jobs that are finished or canceled can be deleted, while jobs that are in certain other states, such as queued, scheduled, or processing, must be canceled first, and then they can be deleted.
The following code example shows a method for deleting a job by checking job states and then deleting when the state is finished or canceled. This code depends on the previous section in this topic for getting a reference to a job: Get a job reference.

	static void DeleteJob(string jobId)
	{
	    bool jobDeleted = false;
	
	    while (!jobDeleted)
	    {
	        // Get an updated job reference.  
	        IJob job = GetJob(jobId);
	
	        // Check and handle various possible job states. You can 
	        // only delete a job whose state is Finished, Error, or Canceled.   
	        // You can cancel jobs that are Queued, Scheduled, or Processing,  
	        // and then delete after they are canceled.
	        switch (job.State)
	        {
	            case JobState.Finished:
	            case JobState.Canceled:
	            case JobState.Error:
	                // Job errors should already be logged by polling or event 
	                // handling methods such as CheckJobProgress or StateChanged.
	                // You can also call job.DeleteAsync to do async deletes.
	                job.Delete();
	                Console.WriteLine("Job has been deleted.");
	                jobDeleted = true;
	                break;
	            case JobState.Canceling:
	                Console.WriteLine("Job is cancelling and will be deleted "
	                    + "when finished.");
	                Console.WriteLine("Wait while job finishes canceling...");
	                Thread.Sleep(5000);
	                break;
	            case JobState.Queued:
	            case JobState.Scheduled:
	            case JobState.Processing:
	                job.Cancel();
	                Console.WriteLine("Job is scheduled or processing and will "
	                    + "be deleted.");
	                break;
	            default:
	                break;
	        }
	
	    }
	}


##Delete an Access Policy

The following code example shows how to get a reference to an access policy based on a policy Id, and then to delete the policy.

	static void DeleteAccessPolicy(string existingPolicyId)
	{
	    // To delete a specific access policy, get a reference to the policy.  
	    // based on the policy Id passed to the method.
	    var policyInstance =
	            from p in _context.AccessPolicies
	            where p.Id == existingPolicyId
	            select p;
	    IAccessPolicy policy = policyInstance.FirstOrDefault();
	
	    policy.Delete();
	
	}
	
