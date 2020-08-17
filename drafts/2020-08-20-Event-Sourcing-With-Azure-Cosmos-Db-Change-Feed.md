---
title: "Event sourcing with Azure Cosmos DB change feed and Azure Functions"
excerpt: "This article presents how to implement event sourcing using Azure Functions and Azure Cosmos DB change feed"
header:
  image: /images/devisland/article40/assets/CosmosChangeFeedEventSourcing1.png
---

<p align="center">
<img src="/images/devisland/article40/assets/CosmosChangeFeedEventSourcing1.png?raw=true" alt="Communication between microservices with Azure Service Bus"/>
</p>

# Introduction

In my [previous article](https://daniel-krzyczkowski.github.io/Event-Sourcing-With-Azure-SQL-And-Entity-Framework-Core/) I described how to implement event sourcing with Azure SQL database and Entity Framework Core. Of course, there are many ways to implement event sourcing in our solutions. In this article, I would like to present how to use Azure Cosmos DB change feed together with Azure Functions to implement event sourcing. If you read my previous article, you probably noticed that we had to update the event log table using source code, every time something is updated in the *Cars* table. We had to write the source code to create a record in the event log table every time we made an update to some record from the *Cars* table. With Azure Cosmos DB change feed and Azure Functions, we can easily react on every change that happens to data in the database without pooling or adding additional code to the logic responsible for adding or updating data. Let's see how to do it.

<p align="center">
<img src="/images/devisland/article40/assets/CosmosChangeFeedEventSourcing3.png?raw=true" alt="Image not found"/>
</p>

# Azure Function with Cosmos DB trigger

Aaa

## Sub-section II

Aaa

```csharp
code...
```


# Summary

Aaa
