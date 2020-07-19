---
title: "Event Sourcing with Azure SQL and Entity Framework Core"
excerpt: "This article presents how to implement event sourcing with Azure SQL database and Entity Framework Core"
header:
  image: /images/devisland/article38/assets/EventSourcingWithEfCoreAndAzureSql1.png
---

<p align="center">
<img src="/images/devisland/article38/assets/EventSourcingWithEfCoreAndAzureSql1.png?raw=true" alt="Event Sourcing with Azure SQL and Entity Framework Core"/>
</p>

# Introduction

Recently I had a chance to read many different articles related to Event Sourcing. This topic is becoming more and more popular especially when talking about microservices. In this article, I would like to present how to add Event Sourcing implementation but before that, just one short summary.

Event Sourcing is just the way to persist state of the entities. There is a lot of confusion nowadays especially when talking about microservices and CQRS. Just to clarify - Event Sourcing has nothing to do with microservices. We can use Event Sourcing in the monolithic systems too. CQRS is not a part of Event Sourcing. Event Sourcing stores each state mutation as a separate record called an event in opposite to state-oriented persistence that only keeps the latest version of the entity state.


# Solution architecture

To demonstrate how to use Azure SQL database together with Entity Framework Core ORM to implement Event Sourcing I decided to start the development of a sample solution called Cars Island. Source code is available on my GitHub.. This project is based on the eShop on containers solution which I found a bit heavy so that is why I decided to create my solution. In the Cars Island solution there are four microservices but we will focus on the *Catalog* microservice and event related to changing car price per day for specific car from the catalog:

<p align="center">
<img src="/images/devisland/article38/assets/EventSourcingWithEfCoreAndAzureSql2.png?raw=true" alt="Image not found"/>
</p>

## Event Log implementation

Aaa

```csharp
code...
```

## Car price per day changed event persistence

Aaa

```csharp
{"CarId":"2605c956-1ab4-4ce9-ad90-559978d754dd","NewPricePerDay":300.0,"OldPricePerDay":200.00,"Id":"9ef1f6ec-ef2e-4ad2-9a15-15284fc94ca7","CreationDate":"2020-07-19T11:22:18.9525308Z"}
```


# Summary

In this article, I presented how to implement Event Sourcing using the Azure SQL database and Entity Framework Core ORM. As we saw Event Sourcing is not so scary. Of course, it depends how complex our solution is but it is good to understand the basics - Event Sourcing is just the way to persist state of the entities. I encourage you to check the source code for the whole solution on [my GitHub.](https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/tree/master/asp-net-core-microservices-with-azure-and-docker)
