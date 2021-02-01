---
title: "Cars Island ASP .NET Core API and Azure Functions - integration with Azure Application Insights - part 6"
excerpt: "This article presents how to integrate ASP .NET Core API and Azure Functions with Azure Application Insights."
header:
  image: /images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights1.jpg
---

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights1.jpg?raw=true" alt="Cars Island ASP .NET Core API and Azure Functions - integration with Azure Application Insights - part 6"/>
</p>


# Introduction

In [my first article](https://daniel-krzyczkowski.github.io/Cars-Island-Car-Rental-On-Azure-Cloud/), I introduced you to Cars Island car rental on the Azure cloud. I created this fake project to present how to use different Microsoft Azure cloud services and how to their SDKs. I also presented what will be covered in the next articles. Here is the next article from the series where I would like to discuss how to integrate with Azure Application Insights in the ASP .NET Core Web API project and Azure Functions.

In the Cars Island solution, Azure Application Insights service is used to monitor health of Azure Function App and Web API including logging errors. Below I present solution architecture:

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights2.png?raw=true" alt="Image not found"/>
</p>


*[Cars Island project is available on my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure)*


# Azure Application Insights setup

## Log Analytics Workspace creation

Before we create the Azure Application Insights service it is good to mention its relationship with Log Analytics. Workspace-based resources support full integration between Application Insights and Log Analytics. You can now choose to send your Application Insights telemetry to a common Log Analytics workspace, which allows you full access to all the features of Log Analytics while keeping application, infrastructure, and platform logs in a single consolidated location.

This is why first I created Log Analytics Workspace:

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights3.PNG?raw=true" alt="Image not found"/>
</p>

As you can see I provided the name for the Log Analytics Workspace (I stuck to the [recommended naming conventions](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming#example-names-integration)). For the pricing tier I selected Pay-As-You-Go option:

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights4.PNG?raw=true" alt="Image not found"/>
</p>


## Azure Application Insights creation

Once Log Analytics Workspace is created, we can create new Azure Application Insights instance and indicate to use the Workspace created above:

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights5.PNG?raw=true" alt="Image not found"/>
</p>

This is it. Once the Azure Application Insights instance is ready we can integrate the application with it using the right SDK. Let's see how to do it below.


# Azure Application Insights .NET SDK integration

## ASP .NET Core Web API integrated with Azure Application Insights

In the Cars Island project, I decided to use [SeriLog](https://serilog.net/) library to implement logic responsible for logging. SeriLog provides additional package (Sink called [Serilog.Sinks.ApplicationInsights](https://github.com/serilog/serilog-sinks-applicationinsights)) which enables sending logs to Azure Application Insights.

Let's start with an explanation of how the initial setup looks like. In the [*SerilogConfigurationSetup.cs*](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.Logging/Configuration/SerilogConfigurationSetup.cs)* file which is located in the [CarsIsland.Logging](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/tree/master/src/web-api/CarsIsland.Logging) there is a setup for the logging.

As you can see there is Azure Application Insights Instrumentation Key passed first to enable sending the logs to Azure cloud. This key can be found in the *Overview* tab for the Azure Application Insights resource, in the Azure portal.

Below fragment of the code below presents the logging setup. As you can see I configured SeriLog to send the logs to the console and the Azure Application Insights. There is also *CustomApplicationInsightsTelemetryConverter* class. It is helpful because with this class you can decide which properties should be included in the logs sent to the Azure Application Insights and also decide which exact value of the log will be assigned to a specific property of the telemetry (like *telemetry.Context.Operation.Id*):


```csharp
    public static class SerilogConfigurationSetup
    {
        public static void WithCarsIslandConfiguration(this LoggerConfiguration loggerConfig,
                                                       IServiceProvider provider, IConfiguration config)
        {
            var instrumentationKey = config["ApplicationInsights:InstrumentationKey"];
            var assemblyName = Assembly.GetEntryAssembly()?.GetName().Name;

            loggerConfig
               .ReadFrom.Configuration(config)
               .Enrich.FromLogContext()
               .Enrich.WithProperty("Assembly", assemblyName)
               .Enrich.FromLogContext()
               .WriteTo.Console(outputTemplate: "[{Timestamp:HH:mm:ss} {Level}] {SourceContext}{NewLine}{Message:lj}{NewLine}{Exception}{NewLine}", theme: AnsiConsoleTheme.Literate)
               .WriteTo.Logger(lc => lc.Filter.ByExcluding(Matching.WithProperty<bool>("Security", p => p))
               .WriteTo.ApplicationInsights(new TelemetryConfiguration { InstrumentationKey = instrumentationKey }, new CustomApplicationInsightsTelemetryConverter()));
        }

        private class CustomApplicationInsightsTelemetryConverter : TraceTelemetryConverter
        {
            public override IEnumerable<ITelemetry> Convert(LogEvent logEvent, IFormatProvider formatProvider)
            {
                foreach (ITelemetry telemetry in base.Convert(logEvent, formatProvider))
                {
                    if (logEvent.Properties.ContainsKey("ErrorId"))
                    {
                        telemetry.Context.Operation.Id = logEvent.Properties["ErrorId"].ToString();
                    }

                    ISupportProperties propTelematry = (ISupportProperties)telemetry;

                    var removeProps = new[] { "MessageTemplate", "ErrorId", "ErrorMessage" };
                    removeProps = removeProps.Where(prop => propTelematry.Properties.ContainsKey(prop)).ToArray();

                    foreach (var prop in removeProps)
                    {
                        propTelematry.Properties.Remove(prop);
                    }

                    yield return telemetry;
                }
            }
        }
    }
```

I encourage you to get familiar with SeriLog. I found it hard to use it with Azure Function Apps (this is why I did not use it in the Mail Notifications Event Handler Function App project).

Now to initialize above configuration, in the [*Program.cs*](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.API/Program.cs) file in the *CreateHostBuilder* method I added *UseSerilog* with injected configuration described above:

```csharp
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                    webBuilder.UseSerilog((provider, context, loggerConfig) =>
                    {
                        loggerConfig.WithCarsIslandConfiguration(provider, Configuration);
                    });
                });
```

Here is the example how to log error in the Web API project:

```csharp
    Log.Error($"Document {blobName} existence cannot be verified - error details: {ex.Message}");
```


## Azure Function App integrated with Azure Application Insights

Mail Notifications Event Handler Function App is also integrated with Azure Application Insights. In this case, I used [Microsoft.Extensions.Logging.ApplicationInsights](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights) library that can be easily integrated with default *ILogger* used in the Azure Function App project. Of course Azure Application Insights Instrumentation Key has to be provided in the configuration.

*ILogger* is used in the [*MailDeliveryService.cs*](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/func-app/CarsIsland.MailSender.Infrastructure/Services/Mail/MailDeliveryService.cs) to log error when email message is not successfully sent by SendGrid service:

```csharp
     _logger.LogError($"SendGrid service returned status code {response.StatusCode} with response: {responseContent}");
```

Now it is time to overview some great capabilities of Azure Application Insights in the Azure portal.

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights6.PNG?raw=true" alt="Image not found"/>
</p>


# Application map

The application map enables you to see the route of sent HHTP requests, and the connection between different components in the system. You can check the response time together with information about the issues. Let's say that list of items is not always properly loaded in your web application. With the application map available in the Application Insights you can verify what is happening when you send the request to your backend systems.

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights7.PNG?raw=true" alt="Image not found"/>
</p>


# Failures

In the Failures tab, you can find information about all failed requests and exceptions that occurred in the application.

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights8.PNG?raw=true" alt="Image not found"/>
</p>

You can easily check what happened, what was the HTTP status code together with details about exception that was thrown. Below there was unauthorized HTTP status returned when connecting to the backend system:

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights9.PNG?raw=true" alt="Image not found"/>
</p>


# Live metrics

With the live metrics tab, you can see all telemetry data like failed requests or incoming requests rate.

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights10.PNG?raw=true" alt="Image not found"/>
</p>


# Search 

The search tab provides an easy way to search all the telemetry data collected in the Azure Application Insights for a specific period.

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights11.PNG?raw=true" alt="Image not found"/>
</p>

It is very helpful because you can display all the details about a specific event that occurred.

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights12.PNG?raw=true" alt="Image not found"/>
</p>


# Availability tests

Under the Availability tab, you can set up a new test and verify whether specific endpoints of the backend system respond as expected - in the specific period. 

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights13.PNG?raw=true" alt="Image not found"/>
</p>

In the new test tab, you can specify test type - in this case, "URL ping test" and provide the URL which should be called by the test. "Test frequency" value provides information about the frequency at which this test will be executed periodically.

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights14.PNG?raw=true" alt="Image not found"/>
</p>

You can also set locations from which requests will be sent to your backend endpoint. In the "success criteria"  you can specify test timeout, expected response HTTP status code, and (optional) response body. You can also set up alert rules which I described below.

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights15.PNG?raw=true" alt="Image not found"/>
</p>


# Alerts

With the alert rules, you can specify when to be notified when some issue occurs. In the example below I have configured an alert for the endpoint used in the availability test. Whenever the average failed locations are greater than or equal to 1 count there will be an email notification sent to the administrator that the backend system is not available.

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights16.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article55/assets/CarsIslandWebApiAzureApplicationInsights17.PNG?raw=true" alt="Image not found"/>
</p>


# Summary

In this article, I described how to set up an Application Insights with Log Analytics Workspace and how to integrate it into the ASP .NET Core Web API application and Azure Function App. Source code of the Cars Island solution is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure) so you can see all implementation details. I also encourage you to watch my course on Pluralsight called [Microsoft Azure Developer: Instrument Solutions for Monitoring and Logging](https://www.pluralsight.com/courses/microsoft-azure-developer-instrument-solutions-monitoring-logging) where I explained how to use Microsoft Azure Application Insights to monitor applications and automatically detect performance anomalies.

If you want to learn more about the Azure Application Insights, you can also check this [official documentation](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview).
