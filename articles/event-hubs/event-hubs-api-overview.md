<properties 
   pageTitle="Event Hubs API Overview"
   description="A summary of some of the key Event Hubs .NET client APIs. "
   services="event-hubs"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="" />
<tags 
   ms.service="event-hubs"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="tbd"
   ms.date="07/10/2015"
   ms.author="sethm" />

# Event Hubs API overview

This article summarizes some of the key Event Hubs .NET client APIs. There are two categories: management and runtime APIs. Runtime APIs consist of all operations needed to send and receive a message. Management operations enable you to manage the Event Hubs entity state by creating, updating, and deleting entities.

Monitoring scenarios span both management and runtime. For detailed reference documentation on the .NET APIs, see the [.NET Class Library](https://msdn.microsoft.com/library/jj933431.aspx) and [EventProcessorHost API](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.aspx) references.

## Management API

To perform the following management operations you must have **Manage** permissions on the Service Bus namespace:

### Create

```
// Create the Event Hub
EventHubDescription ehd = new EventHubDescription(eventHubName);
ehd.PartitionCount = SampleManager.numPartitions;
namespaceManager.CreateEventHubAsync(ehd).Wait();
```

### Update

```
EventHubDescription ehd = await namespaceManager.GetEventHubAsync(eventHubName);

// Create a customer SAS rule with Manage permissions
ehd.UserMetadata = "Some updated info";
string ruleName = "myeventhubmanagerule";
string ruleKey = SharedAccessAuthorizationRule.GenerateRandomKey();
ehd.Authorization.Add(new SharedAccessAuthorizationRule(ruleName, ruleKey, new AccessRights[] {AccessRights.Manage, AccessRights.Listen, AccessRights.Send} )); 
namespaceManager.UpdateEventHubAsync(ehd).Wait();
```

### Delete

```
namespaceManager.DeleteEventHubAsync("event hub name").Wait();
```

## Runtime APIs

### Create publisher

```
// EventHubClient model (uses implicit factory instance, so all links on same connection)
EventHubClient eventHubClient = EventHubClient.Create("event hub name");
```

### Publish message

```
// Create the device/temperature metric
MetricEvent info = new MetricEvent() { DeviceId = random.Next(SampleManager.NumDevices), Temperature = random.Next(100) };
EventData data = new EventData(new byte[10]); // Byte array
EventData data = new EventData(Stream); // Stream 
EventData data = new EventData(info, serializer) //Object and serializer 
    {
       PartitionKey = info.DeviceId.ToString()
    };

// Set user properties if needed
data.Properties.Add("Type", "Telemetry_" + DateTime.Now.ToLongTimeString());

// Send single message async
await client.SendAsync(data);
```

### Create consumer

```
// Create the Event Hub client
EventHubClient eventHubClient = EventHubClient.Create(EventHubName);

// Get the default subscriber group
EventHubSubscriberGroup defaultSubscriberGroup = eventHubClient.GetDefaultSubscriberGroup();

// All messages
EventHubReceiver consumer = await defaultConsumerGroup.CreateReceiverAsync(shardId: index);

// From one day ago
EventHubReceiver consumer = await defaultConsumerGroup.CreateReceiverAsync(shardId: index, startingDateTimeUtc:DateTime.Now.AddDays(-1));
                        
// From specific offset, -1 means oldest
EventHubReceiver consumer = await defaultConsumerGroup.CreateReceiverAsync(shardId: index,startingOffset:-1); 
```

### Consume message

```
var message = await consumer.ReceiveAsync();

// Provide a serializer
var info = message.GetBody<Type>(Serializer)
                                    
// Get a byte[]
var info = message.GetBytes(); 
msg = UnicodeEncoding.UTF8.GetString(info);
```

## Event processor host APIs

These APIs provide resiliency to worker processes that may become unavailable, but distributing shards across available workers.

```
// Checkpointing is done within the SimpleEventProcessor and on a per-consumerGroup per-partition basis, workers resume from where they last left off.
// Use the EventData.Offset value for checkpointing yourself, this value is unique per partition.
string eventHubConnectionString = System.Configuration.ConfigurationManager.AppSettings["Microsoft.ServiceBus.ConnectionString"];
string blobConnectionString = System.Configuration.ConfigurationManager.AppSettings["AzureStorageConnectionString"]; // Required for checkpoint/state

EventHubDescription eventHubDescription = new EventHubDescription(EventHubName);
EventProcessorHost host = new EventProcessorHost(WorkerName, EventHubName, defaultConsumerGroup.GroupName, eventHubConnectionString, blobConnectionString);
            host.RegisterEventProcessorAsync<SimpleEventProcessor>();

// To close
host.UnregisterEventProcessorAsync().Wait();   
```

The [IEventProcessor](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.ieventprocessor.aspx) interface is defined as follows:

```
public class SimpleEventProcessor : IEventProcessor
{
    IDictionary<string, string> map;
    PartitionContext partitionContext;

    public SimpleEventProcessor()
    {
        this.map = new Dictionary<string, string>();
    }

    public Task OpenAsync(PartitionContext context)
    {
        this.partitionContext = context;

        return Task.FromResult<object>(null);
    }

    public async Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
    {
        foreach (EventData message in messages)
        {
            Process messages here
        }
        
        // Checkpoint when appropriate
        await context.CheckpointAsync();

    }


    public async Task CloseAsync(PartitionContext context, CloseReason reason)
    {
        if (reason == CloseReason.Shutdown)
        {
            await context.CheckpointAsync();
        }
    }
}
```

## Next steps

To learn more about Event Hubs scenarios, visit these links:

- [Event Hubs programming guide](event-hubs-programming-guide.md)
- [Event Hubs overview](event-hubs-overview.md)
- [Event Hubs Code Samples](http://code.msdn.microsoft.com/site/search?query=event hub&f[0].Value=event hub&f[0].Type=SearchText&ac=5)

The .NET API references are here:

- [Service Bus and Event Hubs .NET API reference](https://msdn.microsoft.com/library/jj933424.aspx)
- [Event processor host API reference](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.eventprocessorhost.aspx)
- 
