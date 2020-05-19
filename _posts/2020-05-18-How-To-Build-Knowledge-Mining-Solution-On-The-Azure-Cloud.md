---
title: "How to build knowledge mining solution on the Azure cloud"
excerpt: "This article presents how to build knowledge mining solution with Microsoft Azure cloud"
header:
  image: /images/devisland/article35/assets/AzureKnowledgeMining1.PNG
---

<p align="center">
<img src="/images/devisland/article35/assets/AzureKnowledgeMining1.PNG?raw=true" alt="How to build knowledge mining solution on the Azure cloud"/>
</p>

## Introduction

It is worth explaining what knowledge mining is first before we will discover how to build the solution using the Microsoft Azure cloud. Knowledge mining is the process of extracting information from structured and
unstructured content from different sources, using a range of pre-trained and custom AI services like computer vision and natural language
processing. The main target of knowledge mining is to simplify the process of accessing the latent insights contained within structured and unstructured data. Knowledge mining works by orchestrating the overall enrichment pipeline that consists of three main phases:

1. Ingestion
2. Enrichment
3. Exploration & Analysis

Below we are going to explain each phase.


## Ingestion

<p align="center">
<img src="/images/devisland/article35/assets/AzureKnowledgeMining2.PNG?raw=true" alt="Image not found"/>
</p>

Ingestion is the first phase of knowledge mining. This is the process during which we can use structured and unstructured data. Structured data has a defined data model and typically resides in a relational database like Azure SQL. Unstructured data does not have a predefined data model and can come from sources such as NoSQL databases, APIs, blob storage, file stores. Here we can talk about PDF files, images, Word documents, and PowerPoint presentations.

Data ingestion is the process of aggregating raw data (structured or unstructured), from various siloed sources and locations into a persistent, centralized data store.

At the end Ingestion phase, there is also document cracking applied. This is the process of extracting or creating text content from non-text sources, often using optical character recognition (OCR) - this is especially helpful if we want to extract data from images or PDF files.


## Enrichment

<p align="center">
<img src="/images/devisland/article35/assets/AzureKnowledgeMining3.PNG?raw=true" alt="Image not found"/>
</p>

Enrichment is the next phase in the knowledge mining. Once data is ingested and cracked, we can apply AI enrichment on the raw data extracted before to identify patterns, obtain information, and gain understanding from the text contained within images, blobs, and other unstructured data sources. Knowledge mining performs enrichment on individual documents as a sequence of calls to AI models. It is worth mentioning that we can use AI services in the Azure cloud, or build our custom models and use them during the enrichment process.

Most enrichment pipelines start by leveraging pre-trained natural language processing and computer vision AI services available in the Azure cloud:

**Natural language processing**

These services can understand written and spoken human language. These AI services
can interpret sentiment, detect and translate languages, and extract words, key phrases, and the names of
people, locations, and organizations. Language Understanding Intelligent Service or Text Analytics services can be used here as examples.


**Computer vision**

These services can analyze images or videos to detect and classify faces, landmarks, celebrities, or
other objects. They can also caption images and transcribe handwriting. Computer Vision and Custom Vision services can be used here as examples.


## Exploration & Analysis

<p align="center">
<img src="/images/devisland/article35/assets/AzureKnowledgeMining4.PNG?raw=true" alt="Image not found"/>
</p>

The final phase in the knowledge mining is exposing the newly enriched, structured documents, so they are accessible for exploration and analysis. This step might mean adding the documents to a search index or writing them out to a storage location. Exploration is the process of reviewing the added enrichments to learn more about the collected data. The results of enrichment available via search indexes or end-user and line-of-business applications, such as customer relationship management (CRM) or enterprise resource planning (ERP) systems.

Analysis usually refers to the application of analytics tools, such as Power BI, Azure Machine Learning, or Azure Databricks, for exploring and gaining a deeper understanding of the enriched data.



## Knowledge mining solution built on the Azure cloud

<p align="center">
<img src="/images/devisland/article35/assets/AzureKnowledgeMining5.PNG?raw=true" alt="Image not found"/>
</p>

Microsoft Azure cloud provides services to build first-class knowledge mining solution:


**Azure Cognitive Search**

[Azure Cognitive Search](https://docs.microsoft.com/en-us/azure/search/) is a search-as-a-service cloud solution that gives developers APIs and tools for adding a rich search experience. Let me put an example. Imagine that there is a mobile app that you can use to make shopping. You would like to find a specific product so you can use the search box. There are a lot of different products but also maybe you would like to apply filtering (by price for instance). Implementation of own search engine can be time-consuming. In this case, it is worth to use the power of Azure Cognitive Search.

There are two basic approaches used for ingesting data and populating an index in Azure Cognitive Search:

1.Pull data into the index using an Azure Cognitive Search indexer from the supported Azure data source:
 * Azure Blob Storage
 * Azure Data Lake Storage Gen2 (in preview at the moment of writing this article)
 * Azure Table Storage
 * Azure Cosmos DB
 * Azure SQL Database
 * SQL Server on Azure Virtual Machines
 * SQL Managed instances on Azure

<p align="center">
<img src="/images/devisland/article35/assets/AzureKnowledgeMining7.PNG?raw=true" alt="Image not found"/>
</p>

2.Push data into the index programmatically

 The push model relies on custom applications to push documents directly into a search index programmatically.
 Applications can use either the Azure Cognitive Search REST API or the Azure Search SDK for .NET to send data into the index.

<p align="center">
<img src="/images/devisland/article35/assets/AzureKnowledgeMining8.PNG?raw=true" alt="Image not found"/>
</p>


**Azure Cognitive Services**

When using Azure Cognitive Search to build a knowledge mining solution, there is a wide range of pre-trained Microsoft Cognitive Services we can integrate with. The Azure Cognitive Search architecture is extensible and allows assembling an enrichment pipeline from both
[predefined](https://docs.microsoft.com/en-us/azure/search/cognitive-search-predefined-skills) and custom cognitive skills. Custom skills provide a way to insert transformations unique to your content. A custom skill executes independently, applying whatever enrichment step you require. This is the place for Azure Functions to be used.

Here are some examples of Azure Cognitive Services that can be used during the enrichment phase:

Vision APIs:

1. Face API
2. Computer vision API
3. Form recognizer API

Language APIs:

1. Text Analytics API
2. Translator Text API


**Azure Functions**

Azure Functions are ideal for implementing custom skills that are used during the AI enrichment phase. Try to imagine that during the enrichment process, Azure Function can be called and then call other Cognitive Services (like Form Recognizer) to analyze document content and return the results so they can be passed to the Azure Cognitive Search.


On [my GitHub](https://github.com/Daniel-Krzyczkowski/AzureAI/tree/master/src/document-analyzer) I published source code of the custom skill that is built using Azure Function. There is a source code and guide how to build a document analyzer with Function App, Form Recognizer, Logic App, and Cosmos DB.

<p align="center">
<img src="/images/devisland/article35/assets/AzureKnowledgeMining6.png?raw=true" alt="Image not found"/>
</p>


## Real-world example

Together with my team at [Predica](https://www.predicagroup.com/), we delivered knowledge mining solutions for the one of our customers. It was quite big project related to extracing content from the technical requests documents about aircrafts and make it easy to search for potential answers/solutions using web app:

<p align="center">
<img src="/images/devisland/article35/assets/AzureKnowledgeMining9.png?raw=true" alt="Image not found"/>
</p>

We used Azure Cognitive Search together with Azure Cognitive Services. It is worth to mention that we used Form Recognizer in our solution to localize different parts in the form documents. We developed custom skills using Azure Functions. Azure CosmosDB was used to keep additional configuration values. Source files were uploaded to the Azure Blob Storage.


## Summary

Building knowledge mining solution can be challenging, especially if there are a lot of documents in the different formats. Using Azure services it is possible to build such a solution faster. I encourage you to learn more about [Azure Cognitive Search](https://docs.microsoft.com/azure/search/) and [Azure Cognitive Services](https://docs.microsoft.com/en-us/azure/cognitive-services/).
