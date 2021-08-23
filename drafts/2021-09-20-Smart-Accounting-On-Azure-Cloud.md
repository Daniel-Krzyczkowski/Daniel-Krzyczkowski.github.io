---
title: "Smart Accounting on the Azure cloud"
excerpt: "This article explains solutuion architecture and concepts used to implement Smart Accounting project with Azure cloud services."
header:
  image: /images/devisland/article76/assets/SmartAccounting1.png
---

<p align="center">
<img src="/images/devisland/article76/assets/SmartAccounting1.png?raw=true" alt="Smart Accounting on the Azure cloud"/>
</p>


# Introduction

Some time ago I decided to implement solution using microservices architecture style and Microsoft Azure cloud services. Source code of this project is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/Smart-Accounting).

## Business scenario 

Smart Accounting is dedicated solution created for invoices digitalization. Users use web application (Blazor) to upload photos of their invoices. Then, invoice is saved on the Azure Storage Account. After few seconds, invoice is scanned using Azure Form Recognizer, and details from the invoice are saved in the database (Azure Cosmos DB). User can display all details in the web application. This will help users to collect the invoices in one place, searching for specific invoice, and counting different amounts from these invoices. Users can display uploaded photo of the invoice from the invoices library in the web application. This eliminates the problem of storing paper copies of each invoice and makes it easier to find specific invoices.

## Disclaimer

In this solution I decided to use Azure Kubernetes Service (AKS) to host microservices. Please note that the same solutuion could be built with serverless architecture (using Azure Functions). I just wanted to see Kubernetes in action and improve my skills around Kubernetes/Docker technologies.

You could also use Azure Web Apps to host these microservices.


# Solution architecture

Below diagram presents solution architecture and Azure services I used. Please remember that some integrations are simplified (like keeping appropriate security level).

<p align="center">
<img src="/images/devisland/article76/assets/SmartAccounting2.png?raw=true" alt="Image not found"/>
</p>

Let's discuss the purpose of each service used in the above architecture.

## Azure Kubernetes Service (AKS)

Backend microservices are hosted in the Kubernetes cluster in the Azure cloud. Kubernetes is responsible for creating these services and managing their state. If any of these microservices is broken, Kubernetes will re-create it. Below I explained four microservices I developed in this solution:

### File Processor microservice

Microservice responsible for handling file upload. User can upload scanned invoice to be processed by the system. Uploaded files are stored on the Azure Blob Storage. There is also initial record created in the Azure Cosmos DB with details about the file (file name, file URI on the Azure Blob Storage). Once the file is saved, there is new event published to the Azure Service Bus topic.


### Document Analyzer microservice

Microservice responsible for analyzing uploaded invoice files using Form Recognizer service. It receives the event about new file uploaded to the Azure Blob Storage. Once the invoice is scanned, details returned from the Form Recognizer are saved to in the Azure CosmosDB. Once the process is completed, there is new event published to the Azure Service Bus topic.


### Processed Document microservice

Processed documents can be accessed using this microservice. Details about invoices are stored in the Azure CosmosDB, and scanned files are stored on the Azure Blob Storage.


### Notification microservice

Notification service is used to notify users about the scan result in the real time. Once all details are saved in the database by the Document Analyzer microservice , Notification microservice is notified, and publishes new event using Azure SignalR Service to the application.

## Azure Containers Registry

Aaa


## Azure Web App

Azure Web App is used to host Blazor server web application accessed by users.

### Smart Accounting web app

Smart Accounting web application was created for end users. Users can register and sign in to upload their invoices and see scanning results.

## Azure Active Directory B2C (Azure AD B2C)

Aaa


## Azure Storage Account

Aaa

## Azure SendGrid

Aaa


## Azure Cosmos DB

Aaa


## Azure SQL Database

Aaa


## Azure API Management

Aaa


## Azure Form Recognizer

Aaa


## Azure SignalR Service

Aaa


## Azure Service Bus

Aaa


## Azure Key Vault

Aaa


## Azure Application Insights

Aaa



## Azure Monitor

Aaa


# Backend microservices - project structure

## .NET microservices

In this section I would like to discuss backend solution structure in the Visual Studio. You can find the source code in the *src* folder inside [*smart-accounting-backend-services*](https://github.com/Daniel-Krzyczkowski/Smart-Accounting/tree/main/src/smart-accounting-backend-services/src) folder.

<p align="center">
<img src="/images/devisland/article76/assets/SmartAccounting3.PNG?raw=true" alt="Image not found"/>
</p>

Let's discuss the above solution structure and projects which it includes.

### BuildingBlocks folder

*BuildingBlocks* folder contains fours projects:

1. SmartAccounting.Common - contains commonly used classed like API exception handler or common response structure
2. SmartAccounting.EventBus - contains components used to iplmement asynchronous communication amond microservices using Azure Service Bus 
3. SmartAccounting.EventLog - contains components used to implement event history persistence (like storing successfully processed invoice file)
4. SmartAccounting.Logging - contains components used to implement logging with Azure Application Insights

### SmartAccounting.FileProcessor.API

Microservice developed with the ASP .NET 5 Core Web API. This microservice provides endpoint which enables uploading invoice files. Once the invoice file is uploaded, new message is published to the Azure Service Bus to inform Document Analyzer microservice about new file ready for the scanning.

### SmartAccounting.DocumentAnalyzer.API

Microservice developed with the ASP .NET 5 Core Web API. Once there is new message sent from the Azure Service Bus, this microservice calls Azure Form Recognizer to analyze uploaded invoice file stored on the Azure Blob Storage. Once Form Recognizer returns the result, data from the scanned invoice is stored in the Azure Cosmos DB.

### SmartAccounting.Notification.API

Microservice developed with the ASP .NET 5 Core Web API. It is responsible for sending real-time notification to Smart Accounting web application with information that the invoice file was scanned successfully. Azure Signal R Service is used to send real time notifications.

### SmartAccounting.ProcessedDocument.API

Microservice developed with the ASP .NET 5 Core Web API. This microservice provides endpoints to get specific user invoice data or all invoices for specific user.

## docker-compose

Docker compose file is used to build Docker images with all microservices mentioned above. This file also helps running all microservices together on the local development. Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration machine.

Here is *docker-compose.yml* file content from the Smart Accounting solution:

```yml
version: '3.4'

services:
  documentanalyzer.api:
    image: ${DOCKER_REGISTRY-}documentanalyzerapi
    build:
      context: .
      dockerfile: DocumentAnalyzer/SmartAccounting.DocumentAnalyzer.API/Dockerfile


  fileprocessor.api:
    image: ${DOCKER_REGISTRY-}fileprocessorapi
    build:
      context: .
      dockerfile: FileProcessor/SmartAccounting.FileProcessor.API/Dockerfile


  notification.api:
    image: ${DOCKER_REGISTRY-}notificationapi
    build:
      context: .
      dockerfile: Notification/SmartAccounting.Notification.API/Dockerfile


  processeddocument.api:
    image: ${DOCKER_REGISTRY-}processeddocumentapi
    build:
      context: .
      dockerfile: ProcessedDocument/SmartAccounting.ProcessedDocument.API/Dockerfile

```

As you can see, each microservice is built using *Dockerfile*. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Here is [*Dockerfile*](https://github.com/Daniel-Krzyczkowski/Smart-Accounting/blob/main/src/smart-accounting-backend-services/src/DocumentAnalyzer/SmartAccounting.DocumentAnalyzer.API/Dockerfile) for the Document Analyzer microservice:

```yml
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["DocumentAnalyzer/SmartAccounting.DocumentAnalyzer.API/SmartAccounting.DocumentAnalyzer.API.csproj", "DocumentAnalyzer/SmartAccounting.DocumentAnalyzer.API/"]
RUN dotnet restore "DocumentAnalyzer/SmartAccounting.DocumentAnalyzer.API/SmartAccounting.DocumentAnalyzer.API.csproj"
COPY . .
WORKDIR "/src/DocumentAnalyzer/SmartAccounting.DocumentAnalyzer.API"
RUN dotnet build "SmartAccounting.DocumentAnalyzer.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "SmartAccounting.DocumentAnalyzer.API.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "SmartAccounting.DocumentAnalyzer.API.dll"]
```

I encourage you to read more [how Visual Studio builds containerized apps](https://docs.microsoft.com/en-us/visualstudio/containers/container-build?view=vs-2019&WT.mc_id=visualstudio_containers_aka_containerfastmode).


## Kubernetes deployment files

Each backend service is deployed using Kubernetes manifests files. These files can be found in the [*kubernetes*](https://github.com/Daniel-Krzyczkowski/Smart-Accounting/tree/main/src/smart-accounting-backend-services/kubernetes) folder.

We will dicuss these files later in the article.




# Blazor web application - project structure

AAA


# Continuous Integration and Deployment in the Azure DevOps

AAA


# Summary

Aaa
