<properties 
	pageTitle="Consume a Machine Learning web service | Microsoft Azure" 
	description="Once a machine learning service is published, the RESTFul web service that is made available can be consumed either as request-response service or as a batch execution service." 
	services="machine-learning" 
	solutions="big-data" 
	documentationCenter="" 
	authors="bradsev" 
	manager="paulettm" 
	editor="cgronlun" />

<tags 
	ms.service="machine-learning" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.tgt_pltfrm="na" 
	ms.workload="tbd" 
	ms.date="06/29/2015" 
	ms.author="bradsev" />


# How to consume an Azure Machine Learning web service that has been published from a Machine Learning experiment

## Introduction

When published as a web service, Azure Machine Learning experiments provide a REST API that can be consumed by a wide range of devices and platforms. This is because the simple REST API accepts and responds with JSON formatted messages. The Azure Machine Learning portal provides code that can be used to call the web service in R, C#, and Python. But these services can be called with any programming language and from any device that satisfies three criteria:

* Has a network connection
* Has SSL capabilities to perform HTTPS requests
* Has the ability to parse JSON (by hand or support libraries)

This means the services can be consumed from web applications, mobile applications, custom desktop applications and even from within Excel.

[AZURE.INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]  

An Azure Machine Learning web service can be consumed in two different ways, either as a request-response service or as a batch execution service. In each scenario the functionality is provided through the RESTFul web service that is made available for consumption once the experiment has been published. Deploying a Machine Learning web service in Azure with an Azure web service end-point, where the service is automatically scaled based on usage, you can avoid upfront and ongoing costs for hardware resources.

<!-- When this article gets published, fix the link and uncomment
For more information on how to manage Azure Machine Learning web service endpoints using the REST API, see **Azure machine learning web service endpoints**. 
-->

For information about how to create and publish an Azure Machine Learning web service, see [Publish an Azure Machine Learning web service][publish]. For a step-by-step walkthrough of creating a Machine Learning experiment and publishing it, see [Develop a predictive solution by using Azure Machine Learning][walkthrough].

[publish]: machine-learning-publish-a-machine-learning-web-service.md
[walkthrough]: machine-learning-walkthrough-develop-predictive-solution.md


## Request-Response Service (RRS)

A Request-Response Service (RRS) is a low-latency, highly scalable web service used to provide an interface to the stateless models that have been created and published from an Azure Machine Learning Studio experiment.

RRS accepts a single row of input parameters and generates a single row as output. The output row can contain multiple columns.

An example for RRS is validating the authenticity of an application. Hundreds to millions of installations of an application can be expected in this case. When the application starts up, it makes a call to the RRS service with the relevant input. The application then receives a validation response from the service that either allows or blocks the application from performing.


## Batch Execution Service (BES)
 
A Batch Execution Service (BES) is a service that handles high volume, asynchronous, scoring of a batch of data records. The input for the BES contains a batch of records from a variety of sources, such as blobs, tables in Azure, SQL Azure, HDInsight (results of a Hive Query, for example), and HTTP sources. The output for the BES contains the results of the scoring. Results are output to a file in Azure blob storage and data from the storage endpoint is returned in the response.

A BES would be useful when responses are not needed immediately, such as for regularly scheduled scoring for individuals or internet of things (IOT) devices.

## Examples
To show how both RRS and BES work, we use an example Azure Web Service. This service would be used in an IOT (Internet Of Things) scenario. To keep it simple, our device only sends up one value, `cog_speed`, and gets a single answer back. 

There are four pieces of information that are needed to call either the RRS or BES service. This information is readily available from the service pages in [Azure Machine Learning service pages](https://studio.azureml.net) once the experiment has been published. Click on the WEB SERVICES link at the left of the screen and you will see the published services. To find information about a specific service, there are API help page links for both RRS and BES.

1.	The **service API Key**, available on the services main page
2.	The **service URI**, available on the API help page for the chosen service
3.	The expected **API request body**, available on the API help page for the chosen service
4.	The expected **API response body**, available on the API help page for the chosen service

In the two examples below, the C# language is used to illustrate the code needed and the targeted platform is a Windows 8 desktop. 

### RRS Example
On the API help page, aside from the URI, you will input and output definitions and code samples. The API input is called out, for this service specifically, and is the payload of the API call. 

**Sample Request**

	{
	  "Inputs": {
	    "input1": {
	      "ColumnNames": [
	        "cog_speed"
	      ],
	      "Values": [
	        [
	          "0"
	        ],
	        [
	          "1"
	        ]
	      ]
	    }
	  },
	  "GlobalParameters": {}
	}


Similarly, the API response is also called out, again for this service specifically.

**Sample Response**

	{
	  "Results": {
	    "output1": {
	      "type": "DataTable",
	      "value": {
	        "ColumnNames": [
	          "cog_speed"
	        ],
	        "ColumnTypes": [
	          "Numeric"
	        ].
	      "Values": [
	        [
	          "0"
	        ],
	        [
	          "1"
	        ]
	      ]
	    }
	  },
	  "GlobalParameters": {}
	}

Towards the bottom of the page you will find the code examples. Below is the code sample for the C# implementation 
                   
**Sample Code**

	using System;
	using System.Collections.Generic;
	using System.IO;
	using System.Net.Http;
	using System.Net.Http.Formatting;
	using System.Net.Http.Headers;
	using System.Text;
	using System.Threading.Tasks;
	
	namespace CallRequestResponseService
	{
	    public class StringTable
	    {
	        public string[] ColumnNames { get; set; }
	        public string[,] Values { get; set; }
	    }
	
	    class Program
	    {
	        static void Main(string[] args)
	        {
	            InvokeRequestResponseService().Wait();
	        }
	
	        static async Task InvokeRequestResponseService()
	        {
	            using (var client = new HttpClient())
	            {
	                var scoreRequest = new
	                {
	                    Inputs = new Dictionary<string, StringTable> () { 
	                        { 
	                            "input1", 
	                            new StringTable() 
	                            {
	                                ColumnNames = new string[] {"cog_speed"},
	                                Values = new string[,] {  { "0"},  { "1"}  }
	                            }
	                        },
	                    GlobalParameters = new Dictionary<string, string>() { }
	                };
	                
	                const string apiKey = "abc123"; // Replace this with the API key for the web service
	                client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue( "Bearer", apiKey);
	
	                client.BaseAddress = new Uri("https://ussouthcentral.services.azureml.net/workspaces/<workspace id>/services/<service id>/execute?api-version=2.0&details=true");
	                
	                // WARNING: The 'await' statement below can result in a deadlock if you are calling this code from the UI thread of an ASP.Net application.
	                // One way to address this would be to call ConfigureAwait(false) so that the execution does not attempt to resume on the original context.
	                // For instance, replace code such as:
	                //      result = await DoSomeTask()
	                // with the following:
	                //      result = await DoSomeTask().ConfigureAwait(false)

	                HttpResponseMessage response = await client.PostAsJsonAsync("", scoreRequest);
	
	                if (response.IsSuccessStatusCode)
	                {
	                    string result = await response.Content.ReadAsStringAsync();
	                    Console.WriteLine("Result: {0}", result);
	                }
	                else
	                {
	                    Console.WriteLine("Failed with status code: {0}", response.StatusCode);
	                }
	            }
	        }
	    }
	}

### BES Example
On the API help page, in addition to the URI, you will find information about several calls that are available. Unlike the RRS service, the BES service is asynchronous. This means that the BES API is simply queuing up a job to be executed, and the caller polls the job's status to see when it has completed. Here are the operations currently supported for batch jobs:

1. Create (submit) a batch job
1. Start this batch job
1. Get the status or result of the batch job
1. Cancel a running batch job

**1. Create a Batch Execution Job**

When creating a batch job for your Azure Machine Learning service endpoint, one can specify several parameters that will define this batch execution:

* **Input**: represents a blob reference to where the batch job's input is stored.
* **GlobalParameters**: represents the set of global parameters one can define for their experiment. An Azure Machine Learning experiment can have both required and optional parameters that customize the service's execution, and the caller is expected to provide all required parameters, if applicable. These parameters are specified as a collection of key-value pairs.
* **Outputs**: if the service has defined one or more outputs, we allow the caller to redirect any of them to an Azure blob location of their choice. This will allow you to save the service's output(s) in a preferred location and under a predictable name, as otherwise the output blob name is randomly generated. **NOTE** that the service expects the output content, based on its type, to be saved as supported formats:
  - data set outputs: can save as **.csv, .tsv, .arff**
  - trained model outputs: can save as **.ilearner**
  
  Output location overrides are specified as a collection of *<output name, blob reference>* pairs, where the *output name* is the user defined name for a specific output node (also shown on the service's API help page), and *blob reference* is a reference to an Azure blob location where the output is to be redirected to.
  
All these job creation parameters can be optional depending on the nature of your service. For example, services with no input node defined, do not require passing in an *Input* parameter, and the output location override feature is completely optional, as outputs will otherwise be stored in the default storage account that was set up for your Azure Machine Learning workspace. Below, we show a sample request payload, as passed to the REST API, for a service where only the input information is passed in:

**Sample Request**

	{
	  "Input": {
	    "ConnectionString":     
	    "DefaultEndpointsProtocol=https;AccountName=mystorageacct;AccountKey=mystorageacctKey",
	    "RelativeLocation": "/mycontainer/mydatablob.csv",
	    "BaseLocation": null,
	    "SasBlobToken": null
	  },
	  "Outputs": null,
	  "GlobalParameters": null
	}

The response to the batch job creation API is the unique job id that was associated to your job. This id is very important because it provides the only means for you to reference this job in the system for other operations.  
  
**Sample Response**

	"539d0bc2fde945b6ac986b851d0000f0" // The JOB_ID

**2. Start a Batch Execution Job**

Creating a batch job only registers it within the system, and places it in a *Not started* state. To actually schedule the job for execution, you will have to call the **start** API described on the service endpoint's API help page and provide the job id obtained when the job was created.
  
**3. Get the Status of a Batch Execution Job**

You can poll the status of your asynchronous batch job at any time by passing the job's id to the GetJobStatus API. The API response will contain an indicator of the job's current state, as well as the actual results of the batch job if this has completed successfully. In the case of an error, more information about the actual reasons behind the failure are returned in the *Details* property.
 
**Response Payload**

	{
	    "StatusCode": STATUS_CODE,
	    "Results": RESULTS,
	    "Details": DETAILS
	}

*StatusCode* can be one of the following:

* Not started
* Running
* Failed
* Cancelled
* Finished

The *Results* property is populated only if the job has completed successfully (it is **null** otherwise). Upon the job's completion and if the service has at least one output node defined, the results will be returned as a collection of *[output name, blob reference]* pairs, where the blob reference is a SAS read-only reference to the blob containing the actual result. 

**Sample Response**

	{
	    "Status Code": "Finished",
	    "Results":
	    {
	        "dataOutput":
	        {              
	            "ConnectionString": null,
	            "RelativeLocation": "outputs/dataOutput.csv",
	            "BaseLocation": "https://mystorageaccount.blob.core.windows.net/",
	            "SasBlobToken": "?sv=2013-08-15&sr=b&sig=ABCD&st=2015-04-04T05%3A39%3A55Z&se=2015-04-05T05%3A44%3A55Z&sp=r"              
	        },
	        "trainedModelOutput":
	        {              
	            "ConnectionString": null,
	            "RelativeLocation": "models/trainedModel.ilearner",
	            "BaseLocation": "https://mystorageaccount.blob.core.windows.net/",
	            "SasBlobToken": "?sv=2013-08-15&sr=b&sig=EFGH%3D&st=2015-04-04T05%3A39%3A55Z&se=2015-04-05T05%3A44%3A55Z&sp=r"              
	        },           
	    },
	    "Details": null
	}

**4. Cancel a Batch Execution Job**

A running batch job can be cancelled at any time by calling the designated CancelJob API and passing in the job's id. This would be done for various reasons such as that the job is taking too long to complete. 



#### Using the [BES SDK](machine-learning-consume-web-services.md#batch-execution-service-sdk)

The [BES SDK Nugget package](http://www.nuget.org/packages/Microsoft.Azure.MachineLearning/) provides functions that simplify calling BES to score in batch mode. To install the Nuget package, in Visual Studio, go to Tools, then select Nuget Package Manager, and click Package Manager Console. 

AzureML experiments that are published as web services can include web service input modules which means they expect the input to be provided through the web service call in the form of a reference to a blob location. There is also the option of not using a web service input module and using a Reader module instead. In this case, the Reader typically would read from a SQL DB using a query at run time to get the data. Web service parameters can be used to dynamically point to other servers or tables, etc. The SDK supports both of these patterns.

The code sample below demonstrates how you can submit and monitor a batch job against an Azure Machine Learning service endpoint using the BES SDK. Note the comments for details on the settings and calls.

#### **Sample Code**

	// This code requires the Nuget package Microsoft.Azure.MachineLearning to be installed.
	// Instructions for doing this in Visual Studio:
	// Tools -> Nuget Package Manager -> Package Manager Console
	// Install-Package Microsoft.Azure.MachineLearning 
	
	  using System;
	  using System.Collections.Generic;
	  using System.Threading.Tasks;
	  
	  using Microsoft.Azure.MachineLearning;
	  using Microsoft.Azure.MachineLearning.Contracts;
	  using Microsoft.Azure.MachineLearning.Exceptions;
	
	namespace CallBatchExecutionService
	{
	    class Program
	    {
	        static void Main(string[] args)
	        {	            
	            InvokeBatchExecutionService().Wait();
	        }
	
	        static async Task InvokeBatchExecutionService()
	        {
	            // First collect and fill in the URI and access key for your web service endpoint.
	            // These are available on your service's API help page.
	            var endpointUri = "https://ussouthcentral.services.azureml.net/workspaces/YOUR_WORKSPACE_ID/services/YOUR_SERVICE_ENDPOINT_ID/";
	            string accessKey = "YOUR_SERVICE_ENDPOINT_ACCESS_KEY";
	
	            // Create an Azure Machine Learning runtime client for this endpoint
	            var runtimeClient = new RuntimeClient(endpointUri, accessKey);
	
	            // Define the request information for your batch job. This information can contain:
	            // -- A reference to the AzureBlob containing the input for your job run
	            // -- A set of values for global parameters defined as part of your experiment and service
	            // -- A set of output blob locations that allow you to redirect the job's results
	
	            // NOTE: This sample is applicable, as is, for a service with explicit input port and
	            // potential global parameters. Also, we choose to also demo how you could override the
	            // location of one of the output blobs that could be generated by your service. You might 
	            // need to tweak these features to adjust the sample to your service.
	            //
	            // All of these properties of a BatchJobRequest shown below can be optional, depending on
	            // your service, so it is not required to specify all with any request.  If you do not want to
	            // use any of the parameters, a null value should be passed in its place.
	            
	            // Define the reference to the blob containing your input data. You can refer to this blob by its
                    // connection string / container / blob name values; alternatively, we also support references 
                    // based on a blob SAS URI
                    
                    BlobReference inputBlob = BlobReference.CreateFromConnectionStringData(connectionString:                                         "DefaultEndpointsProtocol=https;AccountName=YOUR_ACCOUNT_NAME;AccountKey=YOUR_ACCOUNT_KEY",
                        containerName: "YOUR_CONTAINER_NAME",
                        blobName: "YOUR_INPUT_BLOB_NAME");
                              
                    // If desired, one can override the location where the job outputs are to be stored, by passing in
                    // the storage account details and name of the blob where we want the output to be redirected to.
                    
                    var outputLocations = new Dictionary<string, BlobReference>
                        {
                          {
                           "YOUR_OUTPUT_NODE_NAME", 
                           BlobReference.CreateFromConnectionStringData(                                     connectionString: "DefaultEndpointsProtocol=https;AccountName=YOUR_ACCOUNT_NAME;AccountKey=YOUR_ACCOUNT_KEY",
                                containerName: "YOUR_CONTAINER_NAME",
                                blobName: "YOUR_DESIRED_OUTPUT_BLOB_NAME")
                           }
                        };
	            
	            // If applicable, you can also set the global parameters for your service
	            var globalParameters = new Dictionary<string, string>
	            {
	                { "YOUR_GLOBAL_PARAMETER", "PARAMETER_VALUE" }
	            };
	                
	            var jobRequest = new BatchJobRequest
	            {
	                Input = inputBlob,
	                GlobalParameters = globalParameters,
	                Outputs = outputLocations
	            };
	
	            try
	            {
	                // Register the batch job with the system, which will grant you access to a job object
	                BatchJob job = await runtimeClient.RegisterBatchJobAsync(jobRequest);
	
	                // Start the job to allow it to be scheduled in the running queue
	                await job.StartAsync();
	
	                // Wait for the job's completion and handle the output
	                BatchJobStatus jobStatus = await job.WaitForCompletionAsync();
	                if (jobStatus.JobState == JobState.Finished)
	                {
	                    // Process job outputs
	                    Console.WriteLine(@"Job {0} has completed successfully and returned {1} outputs", job.Id, jobStatus.Results.Count);
	                    foreach (var output in jobStatus.Results)
	                    {
	                        Console.WriteLine(@"\t{0}: {1}", output.Key, output.Value.AbsoluteUri);
	                    }
	                }
	                else if (jobStatus.JobState == JobState.Failed)
	                {
	                    // Handle job failure
	                    Console.WriteLine(@"Job {0} has failed with this error: {1}", job.Id, jobStatus.Details);
	                }
	            }
	            catch (ArgumentException aex)
	            {
	                Console.WriteLine("Argument {0} is invalid: {1}", aex.ParamName, aex.Message);
	            }
	            catch (RuntimeException runtimeError)
	            {
	                Console.WriteLine("Runtime error occurred: {0} - {1}", runtimeError.ErrorCode, runtimeError.Message);
	                Console.WriteLine("Error details:");
	                foreach (var errorDetails in runtimeError.Details)
	                {
	                    Console.WriteLine("\t{0} - {1}", errorDetails.Code, errorDetails.Message);
	                }
	            }
	            catch (Exception ex)
	            {
	                Console.WriteLine("Unexpected error occurred: {0} - {1}", ex.GetType().Name, ex.Message);
	            }
	        }
	    }
	}

 
