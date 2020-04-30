---
title: "Be more efficient developer with Azure Developer Starter Pack"
excerpt: "Azure Developer Starter Pack project with Azure services SDK integrated. This project will help developers to start integration with Azure cloud services like Cosmos DB, Service Bus or Storage Account in their .NET projects"
header:
  image: /images/devisland/article34/assets/AzureDevStarterPack1.png
---

<p align="center">
<img src="/images/devisland/article34/assets/AzureDevStarterPack1.png?raw=true" alt="Be more efficient developer with Azure Developer Starter Pack"/>
</p>

## Introduction

As an Azure Developer, I faced some challenges when implementing projects where integration with multiple services was required. Imagine that you develop Web API and you have to provide functionality for file upload, you have to also insert some data to the database. Each case requires the development and is somehow repeatable. This is why I decided to create the Azure Developer Starter Pack project that is part of the [Azure Developer Templates](https://daniel-krzyczkowski.github.io/AzureDeveloperTemplates/). Source code is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/AzureDeveloperTemplates/tree/master/src/azure-asp-net-core-starter-template).

<p align="center">
<img src="/images/devisland/article34/assets/AzureDevStarterPack2.PNG?raw=true" alt="Image not found"/>
</p>

## What will you find in this project?

In the Starter Project, I used ASP .NET Core Web API template. I integrated Azure SDK of below Azure services:

1. Azure Application Insights
2. Azure Cosmos DB
3. Azure SignalR
4. Entity Framework Core with Azure SQL DB
5. Azure Service Bus
6. Azure Storage Account
7. Azure Event Hub

<p align="center">
<img src="/images/devisland/article34/assets/AzureDevStarterPack3.png?raw=true" alt="Image not found"/>
</p>


## Solution

<p align="center">
<img src="/images/devisland/article34/assets/AzureDevStarterPack4.PNG?raw=true" alt="Image not found"/>
</p>

I tried to apply best practices when implementing this solution and project. Here are some interesting patterns I used:

### System.Threading.Channels for the file upload

If you did not have chance to get familiar with System.Threading.Channels I encourage you to read [this article](https://devblogs.microsoft.com/dotnet/an-introduction-to-system-threading-channels/). I used channels to make file upload more efficient. As you probably know, when client application uploads file to the Web API there is some time when file are stored and reponse is returned. To avoid this waiting on the client site I created "FileProcessingChannel" class and background service class:


```csharp
        [HttpPost]
        [Route("upload")]
        public async Task<IActionResult> Post([FromForm]FilesUpload filesToUpload, CancellationToken cancellationToken)
        {
            if (filesToUpload?.Files == null)
            {
                return BadRequest("No file found to upload");
            }

            long size = filesToUpload.Files.Sum(f => f?.Length ?? 0);
            if (size == 0)
            {
                return BadRequest("No file found to upload");
            }

            foreach (var formFile in filesToUpload.Files)
            {
                var fileTempPath = @$"{Path.GetTempPath()}{formFile.FileName}";

                using (var stream = new FileStream(fileTempPath, FileMode.Create, FileAccess.Write))
                {
                    await formFile.CopyToAsync(stream, cancellationToken);
                }

                var cts = CancellationTokenSource.CreateLinkedTokenSource(cancellationToken);
                cts.CancelAfter(TimeSpan.FromSeconds(3));

                try
                {
                    var fileWritten = await _fileProcessingChannel.AddFileAsync(fileTempPath, cts.Token);

                    if (!fileWritten)
                    {
                        _logger.LogError($"An error occurred when processing file: {formFile.FileName}");
                        return StatusCode(500, $"An error occurred when processing file: {formFile.FileName}");
                    }
                }
                catch (OperationCanceledException) when (cts.IsCancellationRequested)
                {
                    System.IO.File.Delete(fileTempPath);
                    throw;
                }
            }

            return Ok();
        }
```

```csharp
    public class FileProcessingBackgroundService : BackgroundService
    {
        private readonly ILogger<FileProcessingBackgroundService> _logger;
        private readonly FileProcessingChannel _fileProcessingChannel;
        private readonly IStorageService _storageService;

        public FileProcessingBackgroundService(
            ILogger<FileProcessingBackgroundService> logger,
            FileProcessingChannel boundedMessageChannel,
            IStorageService storageService)
        {
            _logger = logger;
            _fileProcessingChannel = boundedMessageChannel;
            _storageService = storageService;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            await foreach (var filePath in _fileProcessingChannel.ReadAllAsync())
            {
                try
                {
                    await using var stream = File.OpenRead(filePath);

                    await _storageService.UploadBlobAsync(stream, Path.GetFileName(filePath));
                    _logger.LogInformation($"File {Path.GetFileName(filePath)} successfully processed");
                }
                finally
                {
                    File.Delete(filePath);
                }
            }
        }
    }
```

### BackgroundService class to listen for messages from Azure Service Bus and Azure Event Hub

The application should listen for the events from the Event Hub and messages from the Service Bus and handle them properly. This is why ASP .NET Core BackgroundService class was used in the project. There are two background services:

#### EventsBackgroundService

```csharp
    public class EventsBackgroundService : BackgroundService
    {
        private readonly IReceivedEventsProcessor _receivedEventsProcessor;
        private readonly ILogger<EventsBackgroundService> _logger;
        public EventsBackgroundService(IReceivedEventsProcessor receivedEventsProcessor,
                                                                    ILogger<EventsBackgroundService> logger)
        {
            _receivedEventsProcessor = receivedEventsProcessor;
            _logger = logger;
        }

        protected override Task ExecuteAsync(CancellationToken stoppingToken) =>
            _receivedEventsProcessor.ExecuteAsync(stoppingToken, (obj) => _logger.LogInformation(obj));
    }
```

#### MessagingBackgroundService

```csharp
    internal class MessagingBackgroundService : BackgroundService
    {
        private readonly IReceivedMessagesProcessor<object> _receivedMessagesProcessor;
        private readonly ILogger<MessagingBackgroundService> _logger;

        public MessagingBackgroundService(IReceivedMessagesProcessor<object> receivedMessagesProcessor,
                                                                    ILogger<MessagingBackgroundService> logger)
        {
            _receivedMessagesProcessor = receivedMessagesProcessor;
            _logger = logger;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            await _receivedMessagesProcessor.ExecuteAsync(stoppingToken, (obj) => _logger.LogInformation((JsonConvert.SerializeObject(obj))));
        }
    }
```

### SignalR Hub class with authentication

Azure SignalR Service enables sending messages in real time. It would be nice to send messages to specific, authenticated users. This is why I implemented Hub with access to the ID of authenticated user:

```csharp
    [Authorize]
    public class RealTimeMessageHub : Hub
    {
        [HubMethodName("direct-message")]
        public async Task SendDirectMessageToUser(string sampleMessageAsJson)
        {
            var sampleMessage = JsonConvert.DeserializeObject<RealTimeMessage>(sampleMessageAsJson);

            sampleMessage.SenderId = new Guid(Context.User.FindFirst(ClaimTypes.NameIdentifier).Value);
            var messageAsJson = JsonConvert.SerializeObject(sampleMessage);

            await Clients.User(sampleMessage.ReceiverId.ToString()).SendAsync(messageAsJson);
        }
    }
```

### Configuration validation

Each service has some configuration parameters. This is why it is worth to validate them on the application startup. For each service I created configuration class together with validation:

```csharp
    public class ApplicationInsightsServiceConfiguration : IApplicationInsightsServiceConfiguration
    {
        public string InstrumentationKey { get; set; }
    }

    public class ApplicationInsightsServiceConfigurationValidation : IValidateOptions<ApplicationInsightsServiceConfiguration>
    {
        public ValidateOptionsResult Validate(string name, ApplicationInsightsServiceConfiguration options)
        {
            if (string.IsNullOrEmpty(options.InstrumentationKey))
            {
                return ValidateOptionsResult.Fail($"{nameof(options.InstrumentationKey)} configuration parameter for the Azure Application Insights is required");
            }

            return ValidateOptionsResult.Success;
        }
    }
```

### Swagger integration with option to provide JWT token

It is much easier to use API with the Swagger API page. This is why I integrated Swagger with option to provide JWT token.

```csharp
    public static class SwaggerCollectionExtensions
    {
        public static IServiceCollection AddSwagger(this IServiceCollection services)
        {
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "Azure Developer Templates - Starter API", Version = "v1" });

                var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
                var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
                c.IncludeXmlComments(xmlPath);

                c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
                {
                    Name = "Authorization",
                    Type = SecuritySchemeType.ApiKey,
                    Scheme = "Bearer",
                    BearerFormat = "JWT",
                    In = ParameterLocation.Header,
                    Description = "JWT Authorization header using the Bearer scheme"
                });
                c.AddSecurityRequirement(new OpenApiSecurityRequirement
                {
                    {
                          new OpenApiSecurityScheme
                            {
                                Reference = new OpenApiReference
                                {
                                    Type = ReferenceType.SecurityScheme,
                                    Id = "Bearer"
                                }
                            },
                            Array.Empty<string>()
                    }
                });
            });

            return services;
        }
    }
```

### Logging with Azure Application Insights

Good logging is very important. This is why I added integration with Azure Application Insights and connected it with Logger class. Filters are configurable:

```csharp
    public static class LoggingServiceCollectionExtensions
    {
        public static IServiceCollection AddLoggingServices(this IServiceCollection services)
        {
            var serviceProvider = services.BuildServiceProvider();
            var azureApplicationInsightsConfiguration = serviceProvider.GetRequiredService<IApplicationInsightsServiceConfiguration>();

            services.AddLogging(builder =>
            {
                builder.AddApplicationInsights(azureApplicationInsightsConfiguration.InstrumentationKey);
                builder.AddFilter<ApplicationInsightsLoggerProvider>("Microsoft", LogLevel.Error);
            });

            services.AddApplicationInsightsTelemetry();
            return services;
        }
    }
```

### More...

I encourage you ti visit Azure Developer Templates repository on [my GitHub](https://github.com/Daniel-Krzyczkowski/AzureDeveloperTemplates/tree/master/src/azure-asp-net-core-starter-template) to review the code. You will find more implementation details, for instance for Azure Cosmos DB integration and Azure Storage Account.


## Summary

In this article, I presented Azure Developer Starter Pack project I developed. I hope this project will be helpful for other Azure Developers and everyone who would like to quickly start building apps using Azure services.
