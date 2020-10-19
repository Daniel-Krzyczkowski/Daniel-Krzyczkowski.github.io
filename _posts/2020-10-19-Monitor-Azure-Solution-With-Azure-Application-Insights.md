---
title: "Monitor Azure Solution With Azure Application Insights"
excerpt: "This article presents how to monitor solution hosted on Azure and troubleshoot issues."
header:
  image: /images/devisland/article44/assets/MonitoringWithApplicationInsights1.png
---

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights1.png?raw=true" alt="Monitor Azure Solution With Azure Application Insights"/>
</p>

# Introduction

There is one important fact to start with. It does not matter if your application solution is hosted on Azure or not, or if it is big or small. Logging and detecting issues quickly are very important. Detecting issues and troubleshooting them can be challenging, especially when in our solution many different components work with each other. In this case, Azure Application Insights can help. In this article, I will present how to use Application Insights to detect issues and prepare availability tests.

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights2.PNG?raw=true" alt="Image not found"/>
</p>

# Application map

The application map enables you to see the route of sent HHTP requests, and the connection between different components in the system. You can check the response time together with information about the issues. Let's say that list of items is not always properly loaded in your web application. With the application map available in the Application Insights you can verify what is happening when you send the request to your backend systems.

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights3.PNG?raw=true" alt="Image not found"/>
</p>


# Failures

In the Failures tab, you can find information about all failed requests and exceptions that occurred in the application.

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights4.PNG?raw=true" alt="Image not found"/>
</p>

You can easily check what happened, what was the HTTP status code together with details about exception that was thrown. Below there was unauthorized HTTP status returned when connecting to the backend system:

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights5.PNG?raw=true" alt="Image not found"/>
</p>


# Live metrics

With the live metrics tab, you can see all telemetry data like failed requests or incoming requests rate.

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights6.PNG?raw=true" alt="Image not found"/>
</p>


# Search 

The search tab provides an easy way to search all the telemetry data collected in the Azure Application Insights for the specific period.

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights7.PNG?raw=true" alt="Image not found"/>
</p>

It is very helpful because you can display all the details about a specific event that occurred.

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights8.PNG?raw=true" alt="Image not found"/>
</p>


# Availability tests

Under the Availability tab, you can set up a new test and verify whether specific endpoints of the backend system respond as expected - in the specific period. 

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights9.PNG?raw=true" alt="Image not found"/>
</p>

In the new test tab, you can specify test type - in this case, "URL ping test" and provide the URL which should be called by the test. "Test frequency" value provides information about the frequency at which this test will be executed periodically.

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights10.PNG?raw=true" alt="Image not found"/>
</p>

You can also set locations from which requests will be sent to your backend endpoint. In the "success criteria"  you can specify test timeout, expected response HTTP status code, and (optional) response body. You can also set up alert rules which I described below.

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights11.PNG?raw=true" alt="Image not found"/>
</p>


# Alerts

With the alert rules, you can specify when to be notified when some issue occurs. In the example below I have configured an alert for the endpoint used in the availability test. Whenever the average failed locations are greater than or equal to 1 count there will be an email notification sent to the administrator that the backend system is not available.

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights12.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights13.PNG?raw=true" alt="Image not found"/>
</p>

# Azure Application Insights integration with ASP .NET Core

Azure Application Insights SDK is available for many different languages like .NET C#, Java, or Phyton. In this article, I would like to present how to integrate Azure Application Insights with the ASP .NET Core Web application. Below I also attach the solution architecture. I want to collect telemetry from my Blazor web application and the ASP .NET Core Web API:

<p align="center">
<img src="/images/devisland/article44/assets/MonitoringWithApplicationInsights14.png?raw=true" alt="Image not found"/>
</p>

For both applications, the integration process is the same. First, we have to add [Microsoft.ApplicationInsights.AspNetCore NuGet package](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore). Then in the "Startup.cs" file, you have to add the below line in the "ConfigureServices" method:


```csharp
 public void ConfigureServices(IServiceCollection services)
 {
     services.AddRazorPages();
     services.AddServerSideBlazor();
     services.AddApplicationInsightsTelemetry();
 }
```

If you want to read more, there is great documentation about how to integrate [ASP .NET Core applications with Azure Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/asp-net-core).


# Summary

In this article, we went through some great features available in the Azure Application Insights service. As you can see this Azure service is very powerful and provides many helpful tools to track issues and troubleshoot them.
