---
title: "Extract Excel file content with Azure Logic Apps and Azure Functions"
excerpt: "This article presents how to use Azure Logic Apps and Azure Functions to extract content from the Excel files"
header:
  image: /images/devisland/article42/assets/ExcelContentExtractionWithAzure1.png
---

<p align="center">
<img src="/images/devisland/article42/assets/ExcelContentExtractionWithAzure1.png?raw=true" alt="Extract Excel file content with Azure Logic Apps and Azure Functions"/>
</p>

# Introduction

Extracting file content in Azure sounds like an easy topic. Let's talk about specific scenario - extracting content from the Excel files. If email is received with Excel file as attachment, we would like to get this content and process it. How to do it? Especially when there is a need to support older file format (xls) and new one (xlsx)? In this article we will go through the implementaton of solution responsible for automated extraction of the contnet from Excel files using Azure Logic Apps and Azure Function Apps. At the end we are goind to store extracted data in the Azure Cosmos DB.

Below diagram presents the flow:

1. Email with Excel file as attachment is received
2. Excel file is stored on the Azure Blog Storage
3. Azure Function is triggered and Excel file is extracted
4. Extracted data is stored in the Azure Cosmos DB

<p align="center">
<img src="/images/devisland/article42/assets/ExcelContentExtractionWithAzure2.png?raw=true" alt="Image not found"/>
</p>


# Azure Logic App implementation

We are going to start with Azure Logic App implementation. In this scenario we want to trigger Logic App when new email with attachment is received. Then we want to store this file on the Blob Storage to make it available for the Azure Function. Here is the flow of the Azure Logic App:

<p align="center">
<img src="/images/devisland/article42/assets/ExcelContentExtractionWithAzure3.PNG?raw=true" alt="Image not found"/>
</p>

Once Excel file is received, we can extract its content using Azure Function App.



# Azure Function App implementation

We need to create Blob Trigger Function App to react always when there is new file created on the Azure Blob Storage.

<p align="center">
<img src="/images/devisland/article42/assets/ExcelContentExtractionWithAzure4.PNG?raw=true" alt="Image not found"/>
</p>

To extract the content, we are going to use *NPOI* library available as [NuGet package](https://www.nuget.org/packages/NPOI/). With this library we can read both - older version of the Excel file (XLS) and the new one (XLSX).


```csharp
code...
```

## Excel file content extraction

Aaa

## Integration with Azure Cosmos DB

Aaa


# Summary

Aaa
