---
title: "Asynchronous communication with Azure Service Bus"
excerpt: "This article presents how to use Azure Service Bus to implement asynchronous communication between services"
header:
  image: /images/devisland/article79/assets/AsynchronousCommunicationWithAzureServiceBus1.png
---

<p align="center">
<img src="/images/devisland/article79/assets/AsynchronousCommunicationWithAzureServiceBus1.png?raw=true" alt="Asynchronous communication with Azure Service Bus"/>
</p>


# Introduction

When architecting application solutions in the Azure cloud it is important to think about important aspects of communication between services. Of course, the most popular way is to use HTTP requests but sometimes it is not enough. Sometimes we have some special requirements like coordinating transactional work that requires a high degree of reliability or load-balancing work across different components of the system. When we want to support these kinds of scenarios it is important to use the right service. Microsoft Azure Service Bus is a fully managed enterprise message broker with message queues and publish-subscribe topics. In this article, I would like to explain some capabilities of the Azure Service Bus and show how it can be used in practice.



# Events vs messages

Before we jump into details related to Azure Service Bus it is important to understand the key difference between events and messages.

## Message

A message is a container decorated with metadata and contains data. A message is produced by a service to be consumed or stored elsewhere. The publisher of the message expects how the consumer handles the message. A contract exists between the two sides. We can put an example here. The publisher publishes a message with information about the new record that should be created in the database by the consumer of the message. In this case, both sides are aware of the message structure and how it should be handled.

## Event

An event is a lightweight notification of a condition or a state change. The publisher of the event does not expect how the event is handled. This is the main difference between messages and events. The consumer of the event decides what to do with the notification. The events are time-ordered and interrelated. The consumer needs the sequenced series of events to analyze what happened. For example, an event notifies consumers that a file was created. It may have general information about the file, but it doesn't have the file itself.

When it comes to Azure Service Bus data is transferred between different applications and services using **messages**. In some application solutions (including my [Smart Acconting](https://github.com/Daniel-Krzyczkowski/Smart-Accounting) solution) you can find implementation for asynchronous communication using Azure Service Bus where suffix *Event* is used, like:

* FileSuccessfullyUploadedIntegrationEvent
* DocumentSuccessfullyAnalyzedIntegrationEvent

These are integration events to inform different components of the solution that something has happened but still when using Azure Service Bus we talk about messages, not events. If we take a look at *FileSuccessfullyUploadedIntegrationEvent* class implementation, we will see that there are some specific details about the uploaded file which have to be passed to another service:

 ```csharp
    internal record FileSuccessfullyUploadedIntegrationEvent : IntegrationEvent
    {
        [JsonPropertyName("userId")]
        public string UserId { get; init; }
        [JsonPropertyName("fileUrl")]
        public string FileUrl { get; init; }
    }
```
We can see that there is *userId*, and *fileUrl* included in the message that will be sent through the Azure Service Bus. In this specific scenario *FileProcessor* service will publish information about uploaded files through the Azure Service Bus. Then *DocumentAnalyzer* service will receive this message and it can analyze file because it knows its URL, and UserID - these details are included in the message content.

To summarize, even if we name the above classes including *IntegrationEvent* suffix, we still use messages. A contract exists between the two services here to make sure that once the file is uploaded, it will be processed correctly.


# Azure Service Bus benefits

Azure Service Bus can be beneficial when we want to support the below scenarios:
* **Messaging** - transfer business data, such as sales or purchase orders, journals, or inventory movements
* **Decouple applications** - improve reliability and scalability of applications and services
* **Load Balancing** - To allow multiple competing consumers to read from a queue at the same time
* **Transactions** - do several operations, all in the scope of an atomic transaction

If you're familiar with other message brokers like Apache ActiveMQ, Service Bus concepts are similar to what you know.


## Concepts and terminology

Let's discuss some concepts and terminology of Azure Service Bus. The data can be any kind of information, including structured data encoded with the common formats such as the following ones: JSON, XML, Apache Avro, Plain Text.

### Namespace

Azure  Service Bus namespace is a container for all messaging components (queues and topics). Multiple queues and topics can be in a single namespace, and namespaces often serve as application containers.

### Queues

A ueue is often used for point-to-point communication. Messages are sent to and received from queues. Queues store messages until the receiving application is available to receive and process them. Messages in queues are ordered and timestamped on arrival. Messages are delivered in pull mode, only delivering messages when requested. One the messages are received by the receiver and marked as completed, it is removed from the queue.

<p align="center">
<img src="/images/devisland/article79/assets/AsynchronousCommunicationWithAzureServiceBus2.png?raw=true" alt="Image not found"/>
</p>

### Topics and subscriptions

Topics and subscriptions are useful in publish/subscribe scenarios. Topics can have multiple, independent subscriptions, which attach to the topic and otherwise work exactly like queues from the receiver side.

<p align="center">
<img src="/images/devisland/article79/assets/AsynchronousCommunicationWithAzureServiceBus3.png?raw=true" alt="Image not found"/>
</p>

A subscriber to a topic can receive a copy of each message sent to that topic. We can define rules on a subscription. A subscription rule has a filter to define a condition for the message to be copied into the subscription and an optional action that can modify message metadata. You can read more about filters and actions in the [official documentation](https://docs.microsoft.com/en-us/azure/service-bus-messaging/topic-filters). Filters and actions can be helpful when for instance:

* We don't want a subscription to receive all messages sent to a topic
* We want to mark up messages with extra metadata when they pass through a subscription

I will show how to use filters in the example I implemented for this article.


# Azure Service Bus setup in Azure portal

I will not describe step by step creation of Queues and Topics with Subscriptions in the Azure portal. You can follow step-by-step tutorials from the official documentation:

* [Use Azure portal to create a Service Bus namespace and a queue](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-quickstart-portal)
* [Use the Azure portal to create a Service Bus topic and subscriptions to the topic](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal)


# Integration in the application's source code

The code sample I am using in this article is available on my GitHub under [this](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/tree/master/asynchronous-communication-with-service-bus) link. Let me briefly explain its structure. There are three projects in the solution:

* TMF.ServiceBusReceiver.API - .NET 5 ASP .NET Web API where we want to receive messages from the Azure Service Bus
* TMF.ServiceBusReceiver.Common - project with common components used to integrate with Azure Service Bus
* TMF.ServiceBusSender.API - .NET 5 ASP .NET Web API where we want to send messages to the Azure Service Bus

<p align="center">
<img src="/images/devisland/article79/assets/AsynchronousCommunicationWithAzureServiceBus5.PNG?raw=true" alt="Image not found"/>
</p>

To integrate with Azure Service Bus I used a new library called *Azure.Messaging.ServiceBus* which is available through NuGet under [this](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/) link.


## Using Queues

Let me start with queues. In the Azure portal, I created one queue called *tmf-queue*. In this section, we will see how to publish messages to the queue, and how to receive them.

<p align="center">
<img src="/images/devisland/article79/assets/AsynchronousCommunicationWithAzureServiceBus4.PNG?raw=true" alt="Image not found"/>
</p>


### Registering dependencies to send messages to the queue and receive them

Let's start with the configuration. In both projects (Sender and Receiver) you will find the [IntegrationServiceCollectionExtensions.cs](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/blob/master/asynchronous-communication-with-service-bus/TMF.ServiceBusSender.API/Core/DependencyInjection/IntegrationServiceCollectionExtensions.cs) file. There is a core configuration for Azure Service Bus.

In the code sample on GitHub you will find comments which indicate required changes to integrate with queues and topics with subscriptions. For brevity I removed comments and parts for the topics with subscriptions in the snippet bellow:

 ```csharp
    internal static class IntegrationServiceCollectionExtensions
    {
        public static IServiceCollection AddIntegrationServices(this IServiceCollection services)
        {
            var eventBusConfiguration = services.BuildServiceProvider().GetRequiredService<IOptions<EventBusConfiguration>>().Value;
            services.AddSingleton<EventBusConfiguration>(eventBusConfiguration);

            services.AddSingleton<IEventBusSubscriptionsManager, InMemoryEventBusSubscriptionsManager>();

            services.AddSingleton(implementationFactory =>
            {
                var serviceBusClient = new ServiceBusClient(eventBusConfiguration.ListenAndSendConnectionString);
                return serviceBusClient;
            });

            services.AddSingleton(implementationFactory =>
            {
                var serviceBusClient = implementationFactory.GetRequiredService<ServiceBusClient>();

                //Creates sender for specific queue:
                var serviceBusSender = serviceBusClient.CreateSender(eventBusConfiguration.QueueName);

                return serviceBusSender;
            });

            services.AddSingleton(implementationFactory =>
            {
                var serviceBusClient = implementationFactory.GetRequiredService<ServiceBusClient>();

                // Creates receiver for specific queue:
                var serviceBusReceiver = serviceBusClient.CreateProcessor(eventBusConfiguration.QueueName,
                                                                          new ServiceBusProcessorOptions
                                                                          {
                                                                              AutoCompleteMessages = false
                                                                          });

                return serviceBusReceiver;
            });

            services.AddSingleton<IEventBus, AzureServiceBusEventBus>();


            return services;
        }
    }
```

There are four crucial classes provided in the *Azure.Messaging.ServiceBus* library which we use in the sample:

* ServiceBusClient - primary interface for developers interacting with the Service Bus client library
* ServiceBusSender is scoped to a particular queue or topic and is created using the ServiceBusClient. The sender allows you to send messages to a queue or topic
* ServiceBusProcessor - can be thought of as an abstraction around a set of receivers. It uses a callback model to allow code to be specified when a message is received and when an exception occurs
* ServiceBusAdministrationClient - the client through which all Service Bus entities can be created, updated, fetched, and deleted. We will use it to manage filters for topics and subscriptions

An important note from the developer perspective is the fact that the ServiceBusClient, senders, receivers, and processors are safe to cache and use as a singleton for the lifetime of the application, which is best practice when messages are being sent or received regularly.

In the above code, we register Service Bus clients which will be used to send and receive messages. As we can see, *ServiceBusClient* instance is created with connection string we have to provide in the app settings. Then *ServiceBusSender* is created - in the constructor we have to pass the name of the queue. Then we create *ServiceBusProcessor* to process incoming messages from the specific queue.

In the end *AzureServiceBusEventBus* instance is registered. We will discuss the structure of this class later in the article. The important fact is that this class provides methods to handle messages from both, Service Bus Queues and Topics with Subscriptions.


## Using Topics and Subscriptions

Topics and subscriptions can be created in the Azure portal I created one topic called *tmf-events* Messages that will be published to this topic. Then we have to create subscriptions so our services can consume these messages.

<p align="center">
<img src="/images/devisland/article79/assets/AsynchronousCommunicationWithAzureServiceBus6.PNG?raw=true" alt="Image not found"/>
</p>

Then I created two subscriptions for my two Web APIs so they can consume messages:

<p align="center">
<img src="/images/devisland/article79/assets/AsynchronousCommunicationWithAzureServiceBus6.PNG?raw=true" alt="Image not found"/>
</p>


Now let's discuss some integration parts to implement sending and receiving messages with topics and subscriptions:

 ```csharp
    internal static class IntegrationServiceCollectionExtensions
    {
        public static IServiceCollection AddIntegrationServices(this IServiceCollection services)
        {
            var eventBusConfiguration = services.BuildServiceProvider().GetRequiredService<IOptions<EventBusConfiguration>>().Value;
            services.AddSingleton<EventBusConfiguration>(eventBusConfiguration);

            services.AddTransient<IIntegrationEventHandler<FileSuccessfullyUploadedIntegrationEvent>, FileSuccessfullyUploadedEventHandler>();
            services.AddSingleton<IEventBusSubscriptionsManager, InMemoryEventBusSubscriptionsManager>();

            services.AddSingleton(implementationFactory =>
            {
                var serviceBusClient = new ServiceBusClient(eventBusConfiguration.ListenAndSendConnectionString);
                return serviceBusClient;
            });

            services.AddSingleton(implementationFactory =>
            {
                var serviceBusSender = serviceBusClient.CreateSender(eventBusConfiguration.TopicName);

                return serviceBusSender;
            });

            services.AddSingleton(implementationFactory =>
            {
                var serviceBusAdministrationClient = new ServiceBusAdministrationClient(eventBusConfiguration
                                                                                        .ListenAndSendConnectionString);
                return serviceBusAdministrationClient;
            });

            services.AddSingleton(implementationFactory =>
            {
                var serviceBusClient = implementationFactory.GetRequiredService<ServiceBusClient>();

                var serviceBusReceiver = serviceBusClient.CreateProcessor(eventBusConfiguration.TopicName,
                                                                         eventBusConfiguration.Subscription,
                                                                         new ServiceBusProcessorOptions
                                                                         {
                                                                             AutoCompleteMessages = false
                                                                         });

                return serviceBusReceiver;
            });

            services.AddSingleton<IEventBus, AzureServiceBusEventBus>();

            var serviceProvider = services.BuildServiceProvider();
            var azureServiceBusEventBus = serviceProvider.GetRequiredService<IEventBus>();

            //Set shouldRemoveDefaultRule to true when using topics and subscriptions:
            azureServiceBusEventBus.SetupAsync(true)
                                   .GetAwaiter()
                                   .GetResult();

            azureServiceBusEventBus.SubscribeAsync<FileSuccessfullyUploadedIntegrationEvent,
                        IIntegrationEventHandler<FileSuccessfullyUploadedIntegrationEvent>>(true)
                        .GetAwaiter().GetResult();


            return services;
        }
    }
```

In the above code we can see that again, we create *ServiceBusSender* but this time as a parameter in the constructor we have to pass topic name. Then we create *ServiceBusAdministrationClient* instance. With this instance, we can manage filters that are applied to specific subscriptions. Let's clarify this here. Subscribers can define which messages they want to receive from a topic. These messages are specified in the form of one or more named subscription rules. Each rule consists of a filter condition that selects particular messages, and optionally contains an action that annotates the selected message. In our case we use *Correlation Filters*, and apply a specific *label* to the message. We will see it in practice when discussing the structure of *AzureServiceBusEventBus* class. You can read more about filters [here](https://docs.microsoft.com/en-us/azure/service-bus-messaging/topic-filters?WT.mc_id=Portal-Microsoft_Azure_ServiceBus).

We have to also create *ServiceBusProcessor* to process messages. When creating a processor we have to pass topic name along with the subscription name.


## Messages handlers

In both scenarios - when we use queue or topics with subscriptions, we can create dedicated handlers to handle these messages. This is why in the above code we use *SubscribeAsync* method with the name of integration event - *FileSuccessfullyUploadedIntegrationEvent*:

 ```csharp
            azureServiceBusEventBus.SubscribeAsync<FileSuccessfullyUploadedIntegrationEvent,
                        IIntegrationEventHandler<FileSuccessfullyUploadedIntegrationEvent>>(true)
                        .GetAwaiter().GetResult();
```

Once the message is received, we want to have a dedicated handler in the source code to handle it.


## Handling messages

Once we register all required dependencies, we can implement source code that will be responsible for sending messages to queues or topics. This is implemented in the [AzureServiceBusEventBus](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/blob/master/asynchronous-communication-with-service-bus/TMF.ServiceBusReceiver.Common/AzureServiceBusEventBus.cs) class. As you probably noticed in this class we use dependencies we added before like *ServiceBusSender*, *ServiceBusProcessor*, and *ServiceBusAdministrationClient*.


### Sending messages

The way we send messages is the same for topics and queues. We create new message with the body which contains event information that happened. Then we send it with *ServiceBusSender.SendMessageAsync* method:


 ```csharp
        public async Task PublishAsync(IntegrationEvent @event)
        {
            var eventName = @event.GetType().Name;
            var jsonMessage = JsonSerializer.Serialize(@event, @event.GetType());
            var body = Encoding.UTF8.GetBytes(jsonMessage);

            var message = new ServiceBusMessage
            {
                MessageId = Guid.NewGuid().ToString(),
                Subject = eventName,
                Body = new BinaryData(body)
            };

            await _serviceBusSender.SendMessageAsync(message);
        }
```

Receiving messages is implemented inside *RegisterSubscriptionClientMessageHandlerAsync* method with *ServiceBusProcessor*:

 ```csharp
        private async Task RegisterSubscriptionClientMessageHandlerAsync()
        {
            _serviceBusReceiver.ProcessMessageAsync += MessageHandler;

            _serviceBusReceiver.ProcessErrorAsync += ErrorHandler;

            await _serviceBusReceiver.StartProcessingAsync();
        }
```

As we can see above, there are two event handlers, one for handling new messages, and the second one for handling errors if the message cannot be processed successfully:

 ```csharp
        private Task ErrorHandler(ProcessErrorEventArgs arg)
        {
            _logger.LogError($"Service Bus Message processing failed: {arg.ErrorSource} {arg.Exception.Message}");
            return Task.CompletedTask;
        }

        private async Task MessageHandler(ProcessMessageEventArgs arg)
        {
            var eventName = arg.Message.Subject;
            if (_subscriptionManager.HasSubscriptionsForEvent(eventName))
            {
                var subscriptions = _subscriptionManager.GetHandlersForEvent(eventName);
                foreach (var subscription in subscriptions)
                {
                    var handler = _serviceProvider.GetService(subscription.HandlerType);
                    if (handler == null) continue;

                    var eventType = _subscriptionManager.GetEventTypeByName(eventName);
                    var messageData = Encoding.UTF8.GetString(arg.Message.Body);

                    var integrationEvent = JsonSerializer.Deserialize(messageData, eventType);
                    var concreteType = typeof(IIntegrationEventHandler<>).MakeGenericType(eventType);
                    await (Task)concreteType.GetMethod("HandleAsync").Invoke(handler, new object[] { integrationEvent });
                    await arg.CompleteMessageAsync(arg.Message);
                }
            }
        }
```

You could probably notice that there is also a dedicated class called [*InMemoryEventBusSubscriptionsManager*](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/blob/master/asynchronous-communication-with-service-bus/TMF.ServiceBusReceiver.Common/InMemoryEventBusSubscriptionsManager.cs). This class is used to register handlers for messages. In this specific scenario, we keep event handlers registrations in memory. In the production scenarios the good idea is to store this information somewhere else like in the database. Just to clarify. When talking about subscriptions in this context, **we are not talking about subscriptions to topics in the Service Bus**. In this context, we talk about subscriptions of message handlers to specific messages types that can be sent through the Azure Service Bus queue or topic.

In this specific scenario I created [*FileSuccessfullyUploadedIntegrationEvent*](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/blob/master/asynchronous-communication-with-service-bus/TMF.ServiceBusReceiver.API/Application/IntegrationEvents/FileSuccessfullyUploadedIntegrationEvent.cs) class which keep information about the uploaded file and the owner of the file (user ID):

 ```csharp
    internal record FileSuccessfullyUploadedIntegrationEvent : IntegrationEvent
    {
        [JsonPropertyName("userId")]
        public string UserId { get; init; }
        [JsonPropertyName("fileUrl")]
        public string FileUrl { get; init; }
    }
```

There is also event handler for this specific integration event - [*FileSuccessfullyUploadedEventHandler*](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/blob/master/asynchronous-communication-with-service-bus/TMF.ServiceBusReceiver.API/Application/IntegrationEvents/EventHandler/FileSuccessfullyUploadedEventHandler.cs):


```csharp
    internal class FileSuccessfullyUploadedEventHandler : IIntegrationEventHandler<FileSuccessfullyUploadedIntegrationEvent>
    {
        private readonly ILogger<FileSuccessfullyUploadedEventHandler> _logger;

        public FileSuccessfullyUploadedEventHandler(ILogger<FileSuccessfullyUploadedEventHandler> logger)
        {
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        public async Task HandleAsync(FileSuccessfullyUploadedIntegrationEvent @event)
        {
            if (!string.IsNullOrEmpty(@event.FileUrl)
                                            && !string.IsNullOrEmpty(@event.UserId))
            {
                _logger.LogInformation($"Received new event with user ID: {@event.UserId} and file URL: {@event.FileUrl}");
            }
        }
    }
```

In the *HandleAsync* method we can proceed with message processing.


Now let's get back to filters for subscriptions. By default - there is always a default rule applied to subscription on a specific topic. It means that every message that is sent to a specific topic will be delivered to each subscriber. In real-world scenarios, we want to avoid such situations. We want to subscribe to messages sent to specific topics but we want to also filter these messages out to make sure that we will receive the message that is important for us. In this case when we subscribe to Azure Service Bus topic, we want to remove the default subscription rule. This is done in the [*RemoveDefaultRuleAsync*](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/blob/81d5b9cb5e380f0e0b2c5be735d80bb4cc2704ba/asynchronous-communication-with-service-bus/TMF.ServiceBusReceiver.Common/AzureServiceBusEventBus.cs#L166) method:


```csharp
 await _serviceBusAdministrationClient.DeleteRuleAsync(_eventBusConfiguration.TopicName,
                                                                      _eventBusConfiguration.Subscription,
                                                                      RuleProperties.DefaultRuleName);
```

With the above code we will remove the default rule from the Azure Service Bus subscription and we will not receive all the messages by default. Then we have to specify which messages we want to receive. To do it inside the [*SubscribeAsync*](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/blob/81d5b9cb5e380f0e0b2c5be735d80bb4cc2704ba/asynchronous-communication-with-service-bus/TMF.ServiceBusReceiver.Common/AzureServiceBusEventBus.cs#L80) method we apply *CorrelationRuleFilter*:

```csharp
        await _serviceBusAdministrationClient.CreateRuleAsync(_eventBusConfiguration.TopicName,
                                          _eventBusConfiguration.Subscription,
                                               new CreateRuleOptions
                                                    {
                                                      Filter = new CorrelationRuleFilter
                                                        {
                                                          Subject = eventName
                                                        },
                                                          Name = eventName
                                                      });
```

With the above code, we can state that we want to receive messages sent to the topic but only when these messages have specific event's name assigned to *Subject* property of the message. In my sample I only subscribe to messages where *Subject* equals *FileSuccessfullyUploadedIntegrationEvent*:

```csharp
           azureServiceBusEventBus.SubscribeAsync<FileSuccessfullyUploadedIntegrationEvent,
                        IIntegrationEventHandler<FileSuccessfullyUploadedIntegrationEvent>>(true)
                        .GetAwaiter().GetResult();
```


# Test sending messages to queues and topics

If we launch these two APIs, we can call [*PublishEvent*](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/blob/81d5b9cb5e380f0e0b2c5be735d80bb4cc2704ba/asynchronous-communication-with-service-bus/TMF.ServiceBusSender.API/Controllers/EventsController.cs#L25) endpoint from the Sender API. Then in the console of the Receiver API, we should see the messages received:

<p align="center">
<img src="/images/devisland/article79/assets/AsynchronousCommunicationWithAzureServiceBus8.PNG?raw=true" alt="Image not found"/>
</p>


# Summary

In this article, we discussed the difference between Azure Service Bus queues and topics and how to integrate with Azure Service Bus using *Azure.Messaging.ServiceBus* library. I hope that the code sample I created will be helpful for you to integrate with Azure Service Bus in your solutions.
