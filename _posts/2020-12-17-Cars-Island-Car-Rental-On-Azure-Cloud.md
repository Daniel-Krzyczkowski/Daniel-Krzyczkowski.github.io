---
title: "Cars Island Car Rental on Azure cloud"
excerpt: "This article presents an introduction to the implementation of Car Island fake car rental solution built with Microsoft Azure cloud."
header:
  image: /images/devisland/article50/assets/CarsIslandOnAzurePart1-1.png
---

<p align="center">
<img src="/images/devisland/article50/assets/CarsIslandOnAzurePart1-2.png?raw=true" alt="Cars Island Car Rental on Microsoft Azure cloud"/>
</p>


# Introduction

Some time ago I have realized that I have collected some good resources from different projects that I can share with the Azure community. I have decided that I want to create a fake solution, implement it, and include an explanation about planning, implementation, and release process details. In this first article, I would like to introduce you to fake Cars Island car rental solution built on the Microsoft Azure cloud. There will be a series of an article describing not only the implementation details but also usage and configuration of Azure services together with best DevOps practices used (including backlog creation in Azure DevOps). Below you can find an explanation of what will be included in each of the articles. 

Of course, the whole solution will be published on my GitHub soon, together with the next article.

<p align="center">
<img src="/images/devisland/article50/assets/CarsIslandOnAzurePart1-3.PNG?raw=true" alt="Image not found"/>
</p>

**Important**

Just to make it clear, this project presents some concepts and patterns which can be used to implement solutions with the Azure cloud. Some aspects were simplified to make it possible to work without additional dependencies (that is why you will not find payment functionality for instance).

# Cars Island car rental functionalities

There is a web portal where customers can see all available cars for rent in the Cars Island car rental company. Once the customer signs in, there is an option to reserve a specific car. Once the reservation is completed, there is a confirmation email sent. Customers can also send a new enquiry (anonymously).

<p align="center">
<img src="/images/devisland/article50/assets/CarsIslandOnAzurePart1-3.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article50/assets/CarsIslandOnAzurePart1-4.PNG?raw=true" alt="Image not found"/>
</p>


# Cars Island car rental architecture

In this section, you will find an explanation of Azure components used in the solution.

<p align="center">
<img src="/images/devisland/article50/assets/CarsIslandOnAzurePart1-1.png?raw=true" alt="Image not found"/>
</p>


## Azure Active Directory B2C

Azure Active Directory B2C is an identity service in the Azure cloud that enables user authentication and management. Implementing own identity service can be challenging. Try to think about data storage, secure connections, or token generation and validation. With Azure AD B2C, adding user authentication is much easier. In the Cars Island solution users can create accounts and login to access some of the functionalities. What is more - with Azure AD B2C, login, registration, password reset, or profile edit pages can be customized and branded (like with the company logo or background). You do not have to implement login, or registration views yourself.


## Azure Key Vault

Security is a very important aspect of every project. Secrets and credentials should be stored in the secure store. This is why Azure Key Vault is used in the Cars Island solution. Parameters like connection string to the database or storage key are stored in the Azure Key Vault instance.


## Azure Application Insights

Tracking issues in cloud solutions can be challenging. Collecting logs and detecting bugs can be hard. This is why it is good to use an Application Performance Management service like Azure Application Insights. With this Azure cloud service, we can log all events and errors that occure in our solution. Azure Application Insights provides SDKs in many languages (like C# oraz Java) so we can easily integrate them with our application. All logs are then available in the Azure portal, where rich dashboards are displayed with collected log data.


## Azure Web Apps with App Service Plans

Hosting web applications in the Azure cloud is much easier with Azure Web Apps. Cars Island Web portal is written with Blazor framework and Cars Island API is written with ASP .NET Core. These web applications are hosted using Azure Web Apps. Azure App Service Plans provide a way to scale in and out, up and down so you can apply automatic scale when there is higher load and traffic. With Azure Web Apps we can also use custom SSL certificates.


## Azure Function Apps

Azure Function Apps are serverless services available in the Azure cloud. They are ideal to be used as event handlers for processing events. It is important to mention the cost - you only pay for this service once it is executed. Up to 1 million executions is for free. In the Cars Island solution, Azure Function App was used to handle events related to sending car's reservation confirmation emails once custom complete reservation in the web portal. This Function App is triggered once there is a new message in the Azure Service Bus queue.


## Azure Service Bus

Azure Service Bus service is a cloud messaging service. With Azure Service Bus we can build reliable and elastic cloud apps with messaging. In the Cars Island solution, Azure Service Bus queues were used to queue car's reservation confirmations to send emails to customers. Once car's reservation is completed in the web portal, information is passed to the API and saved in the database. After this process, there is new message sent to the queue. Then Azure Function is triggered and new email is sent using Azure SendGrid Service.


## Azure SendGrid

Azure SendGrid service enables sending customized emails. It is great because we can create email templates but also it provides SDK that we can use to implement emails sending in the source code. Up to 25.000 emails, it is free.


## Azure Cosmos DB

Azure Service Bus is a globally distributed database available in the Azure cloud. Data about all cars and reservations is stored in this database in the Cars Island solution.


## Azure Storage Account

Azure Storage Account is one of the oldest services available in the Azure cloud. It provides an easy way to store different kind of files using Blob Storage. In the Cars Island solution, it is used to store car images that are then displayed in the web portal.


## Azure API Management

Azure API Management is a service that works as a gateway to different APIs behind it. With Azure API Management you can secure your APIs. It provides a different kind of policies so for instance we can implement throttling or validate tokens. In the Cars Island solution, it was used to protect access to Cars Island API.


# What will be included in the articles series?


## 1. Introduction. General overview of solution architecture and Azure components used. Decisions behind using specific services. 

- Cars Island API structure – clean architecture. Explanation of the project's structure and functionalities 

- Cars Island Web App – project structure and functionalities

 

## 2. Cars Island ASP .NET Core API - secured by Azure AD B2C 

- Information about integration with Azure AD B2C 

- Usage of Microsoft Identity Web library 

- Secured endpoints 

- User claims access 

- Configuration of the application in the Azure AD B2C apps panel in the Azure portal 

 

## 3. Cars Island ASP .NET Core API - integration with Azure Cosmos DB 

- Explanation of Azure Cosmos DB functionalities (including multi-region read and write) 

- Azure Cosmos DB integration with the new Azure SDK 

- Configuration and implementation in the source code 

 

## 4. Cars Island ASP .NET Core API - integration with Azure Storage Account 

- Explanation of Azure Cosmos DB functionalities (including multi-region read and write) 

- Azure Storage integration with the new Azure SDK 

- Configuration and implementation in the source code 

 

## 5. Cars Island ASP .NET Core API - integration with Azure Service Bus Queue 

- Explanation of Azure Service Bus functionalities (messages, events, topics, queues) 

- Azure Service Bus integration with the new Azure SDK 

- Configuration and implementation in the source code 

 

## 6. Cars Island ASP .NET Core API - integration with Azure Application Insights 

- Explanation of Azure Application Insights functionalities 

- SeriLog integration with Azure Application Insights 

- Configuration and implementation in the source code 

 

## 7. Cars Island Function Apps - integration with Azure SendGrid and Azure Service Bus queues 

- Explanation of Azure SendGrid functionalities 

- Azure SendGrid integration with SDK 

- Configuration and implementation in the source code 

- Azure Service Bus queue trigger setup 

 

## 8. Cars Island Function Apps - integration with Microsoft Graph SDK and Azure AD B2C 

- Configuration and implementation in the source code using Microsoft Graph C# SDK 

- Communication with Azure AD B2C using Microsoft Graph C# SDK 

 

## Controlled access to Cars Island ASP .NET Core API with Azure API Management 

- Azure API Management functionalities and configuration 

- Policies used (throttling and others) 

 

## 9. Cars Island Blazor Web App - secured by Azure AD B2C 

- Information about integration with Azure AD B2C 

- Usage of Microsoft Identity Web library 

- Secured endpoints 

- User claims access 

- Configuration of the application in the Azure AD B2C apps panel in the Azure portal 

- Token acquisition to call downstream API 

 

## 10. Cars Island - Azure Infrastructure as a Code  

- ARM templates structure  

- Using ARM templates with Visual Studio Code 

 

## 11. Cars Island – backlog structure and definition 

- Backlog definition in the Azure DevOps 

- Statuses 

- Queries 

- Dashboards 

 

### 12. Cars Island – DevOps practices for the Azure Infrastructure 

- Creating a release pipeline in the Azure DevOps using YAML files 

- Using environments 

- Using approvals and gates 

- Using variable groups 

 

### 13. Cars Island – DevOps practices for the application release 

- Creating a release pipeline in the Azure DevOps using YAML files 

- Using environments 

- Using approvals and gates 

- Using variable groups 


# Summary

In this article, I introduced the Cars Island fake car rental solution built with Microsoft Azure cloud services. In the next articles, I am going to explain implementation details and share the GitHub repository.
