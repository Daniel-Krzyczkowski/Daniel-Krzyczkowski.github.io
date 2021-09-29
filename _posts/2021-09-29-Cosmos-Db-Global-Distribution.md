---
title: "How multiple writes and reads are handled in Azure Cosmos DB"
excerpt: "This article presents how to enable geo-replication and multiple writes regions in the Azure Cosmos DB"
header:
  image: /images/devisland/article76/assets/CosmosDbGlobalDistribution1.png
---

<p align="center">
<img src="/images/devisland/article76/assets/CosmosDbGlobalDistribution1.png?raw=true" alt="How to handle multiple writes and reads in Azure Cosmos DB"/>
</p>


# Introduction

Azure Cosmos DB is a fully managed NoSQL database for modern app development. Today's applications are required to be highly responsive and always online. To achieve low latency and high availability, instances of these applications need to be deployed in data centers that are close to their users. In this article, we are going to see how to enable geo-replication together with multiple writes in the Azure Cosmos DB. In this article I use ASP .NET Core API with Azure Cosmos DB SDK V3 integrated to connect with Azure Cosmos DB. First, let's talk about some concepts behind geo-replication and multiple write regions.


# Geo-replication in the Azure Cosmos DB

Azure Cosmos DB enables geo-replication to multiple regions. By default, the Azure Cosmos DB account is created with one region which is used for read and write operations. If we open *Replicate data globally* tab we can see the default region for reads and writes operations:

<p align="center">
<img src="/images/devisland/article76/assets/CosmosDbGlobalDistribution2.PNG?raw=true" alt="Image not found"/>
</p>

During the creation of the new Azure Cosmos DB account, we can enable *Geo-Redundancy*, and *Multi-region Writes* options:

<p align="center">
<img src="/images/devisland/article76/assets/CosmosDbGlobalDistribution3.PNG?raw=true" alt="Image not found"/>
</p>

When we select the primary region where our Azure Cosmos DB should be deployed, and enable *Geo-Redundancy* option, there will be another paired region added automatically during creation. Let me put an example. If I create my account with the West Europe region, Azure will automatically assign second paired region which is North Europe in this case:

<p align="center">
<img src="/images/devisland/article76/assets/CosmosDbGlobalDistribution4.PNG?raw=true" alt="Image not found"/>
</p>

*Multi-region Writes* option enabled will mark above read regions as write regions too.

Then, in our application source code, we can decide which region will be our primary region for reads and writes - we will see it later in this article.

## Possible scenarios 

Before we jump into the code, let's talk about different kinds of scenarios related to handling issues with regions and write/read operations.

### Single-region accounts

When we use only one region in our Azure Cosmos DB (let's say West Europe) for handling reads and writes operations, if there is a region outage, applications will experience loss of read and write availability. It just means that our applications will not be able to read data and write data when there is an issue with the region. There is no way to failover.

### Multiple read regions

Multi-region accounts will experience different behaviors depending on the following table provided in the [Azure Cosmos DB documentation](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/cosmos-db/high-availability.md). Please note that behavior is different when we have manual failover enabled or automatic failover enabled for our Azure Cosmos DB account:

| Write regions | Automatic failover | What to expect | What to do |
| -- | -- | -- | -- |
| Single write region | Not enabled | In case of an outage in a read region, all clients will redirect to other regions. No read or write availability loss. No data loss. <p/> In case of an outage in the write region, clients will experience write availability loss. If a strong consistency level is not selected, some data may not have been replicated to the remaining active regions. This depends on the consistency level selected as described in [this section](consistency-levels.md#rto). If the affected region suffers permanent data loss, unreplicated data may be lost. <p/> Cosmos DB will restore write availability automatically when the outage ends. | During the outage, ensure that there are enough provisioned RUs in the remaining regions to support read traffic. <p/> Do *not* trigger a manual failover during the outage, as it will not succeed. <p/> When the outage is over, re-adjust provisioned RUs as appropriate. |
| Single write region | Enabled | In case of outage in a read region, all clients will redirect to other regions. No read or write availability loss. No data loss. <p/> In case of an outage in the write region, clients will experience write availability loss until Cosmos DB automatically elects a new region as the new write region according to your preferences. If a strong consistency level is not selected, some data may not have been replicated to the remaining active regions. This depends on the consistency level selected as described in [this section](consistency-levels.md#rto). If the affected region suffers permanent data loss, unreplicated data may be lost. | During the outage, ensure that there are enough provisioned RUs in the remaining regions to support read traffic. <p/> Do *not* trigger a manual failover during the outage, as it will not succeed. <p/> When the outage is over, you may move the write region back to the original region, and re-adjust provisioned RUs as appropriate. Accounts using SQL APIs may also recover the non-replicated data in the failed region from your [conflicts feed](how-to-manage-conflicts.md#read-from-conflict-feed). |
| Multiple write regions | Not applicable | No read or write availability loss. <p/> Recently updated data in the failed region may be unavailable in the remaining active regions. Eventual, consistent prefix, and session consistency levels guarantee a staleness of <15mins. Bounded staleness guarantees less than K updates or T seconds, depending on the configuration. If the affected region suffers permanent data loss, unreplicated data may be lost. | During the outage, ensure that there are enough provisioned RUs in the remaining regions to support additional traffic. <p/> When the outage is over, you may re-adjust provisioned RUs as appropriate. If possible, Cosmos DB will automatically recover non-replicated data in the failed region using the configured conflict resolution method for SQL API accounts, and Last Write Wins for accounts using other APIs. |

In this article, we will use an Azure Cosmos DB account with three regions, and multiple-region writes options enabled:

* West Europe
* North Europe
* Central US

<p align="center">
<img src="/images/devisland/article76/assets/CosmosDbGlobalDistribution5.PNG?raw=true" alt="Image not found"/>
</p>


### Manual failover vs automatic failover when multi-region writes option disabled

As you can see all three regions are marked as read and write enabled regions. Let's clarify reads and writes a little bit. When we have multiple regions enabled but the multiple-region writes option is disabled, there will be only one write region, which will accept all the write requests from all application instances (we can have multiple instances in Europe for instance, in the United States, and India). All read operations will be redirected to the current write region.

<p align="center">
<img src="/images/devisland/article76/assets/CosmosDbGlobalDistribution6.PNG?raw=true" alt="Image not found"/>
</p>

When there is a region outage, all read operations will be redirected to the second region selected by Azure Cosmos DB as the most optimal endpoint to perform write and read operations.

If we do not enable *Automatic failover* option, we can initialize failover manually, using *Manual failover* tab. There we can select which region will become the new write region for all write operations:

<p align="center">
<img src="/images/devisland/article76/assets/CosmosDbGlobalDistribution7.PNG?raw=true" alt="Image not found"/>
</p>

If we want to enable *Automatic failover* option, we can specify the list of the regions which will become write regions in case of region outage. As we can see, these regions are in the specific order with specific priority assigned.

<p align="center">
<img src="/images/devisland/article76/assets/CosmosDbGlobalDistribution8.PNG?raw=true" alt="Image not found"/>
</p>

In the above scenario, first North Europe will be selected as a new write region, and if it is not available too, Central US will be selected. 


### Failover when multi-region writes option enabled

When we enable the multi-writes option, failover is instantiated automatically and there is no way to do failover manually:

<p align="center">
<img src="/images/devisland/article76/assets/CosmosDbGlobalDistribution9.PNG?raw=true" alt="Image not found"/>
</p>

In this case all regions are marked as read and write regions. Now it is time to talk about application source code, how it is handled in the code using Azure Cosmos DB SDK.


# Global data distribution with Azure Cosmos DB SDK

In this article I will use .NET 5 ASP .NET Core API application with [Microsoft.Azure.Cosmos v3](https://www.nuget.org/packages/Microsoft.Azure.Cosmos) NuGet package added.

## Read regions selection

First of all, it is worth mentioning that handling multiple read regions does not require any changes in the application source code. In this case, the SDK automatically directs both reads and writes to the current write region. If there is an outage of the current region and we have automatic failover enabled, Azure Cosmos DB will select the next region to become a write region. Azure Cosmos DB SDK has resiliency mechanisms implemented so it will detect that there is a new region selected so all read and write operations will be sent to this region.

We can also specify the list with preferred read regions when initializing a connection using the SQL SDKs. Azure Cosmos DB SDKs accept an optional parameter *PreferredLocations* that is an ordered list of Azure regions. The SDK will automatically send all writes to the current write region. All reads will be sent to the first available region in the preferred locations list. If the request fails, the client will fail down the list to the next region. The SDK will only attempt to read from the regions specified in preferred locations. If the Azure Cosmos account is available in four regions, but the client only specifies two read(non-write) regions within the *PreferredLocations*, then no reads will be served out of the read region that is not specified in *PreferredLocations*.

Here is the code sample of how to initialize *CosmosClient* with *PreferredLocations* list of read regions:

 ```csharp
            var cosmosEndpointUrl = "";
            var cosmosPrimaryKey = "";
            var cosmosClient = new CosmosClient(cosmosEndpointUrl, cosmosPrimaryKey, new CosmosClientOptions
                {
                    ApplicationPreferredRegions = new List<string> { Regions.WestEurope, Regions.NorthEurope }
                });
```

As we can see, I specify the list of regions, in this case West Europe, and North Europe. If there will be a problem with West Europe, North Europe will become a new region for reads. If the *PreferredLocations* property is not set, all requests will be served from the current write region.

There is also a possibility to directly indicate the region where our application is deployed. With *ApplicationRegion* property, the SDK automatically populates the preferred locations based on the current region that the client is running in. Here is the code example:

 ```csharp
            var cosmosEndpointUrl = "";
            var cosmosPrimaryKey = "";

            var cosmosClient = new CosmosClient(cosmosEndpointUrl, cosmosPrimaryKey, new CosmosClientOptions
            {
                ApplicationRegion = Regions.WestEurope
            });
```

## Handling multi-write regions

If we have multi-write region option enabled for our Azure Cosmos DB account, we can specify in the code which region will be our primary region for read and write operations. Please note that in this case, we cannot have both declared, *ApplicationRegion*, and *ApplicationPreferredRegions* specified. This is a scenario where we want to directly say that when our application is deployed in a specific region we want to perform read and write operations using the closest Azure Cosmos DB region we specified in the Azure portal when we configured our account.

 ```csharp
            var cosmosEndpointUrl = "";
            var cosmosPrimaryKey = "";

            var cosmosClient = new CosmosClient(cosmosEndpointUrl, cosmosPrimaryKey, new CosmosClientOptions
            {
                ApplicationRegion = Regions.WestEurope
            });
```

In the above scenario, SDK will send all read and writes operations to the West Europe region. When there is *ApplicationRegion* property specified in the code, the *PreferredLocations* property is automatically filled out based on the geo-proximity from the location passed in. If a new region is later added to the Azure Cosmos DB account, the application does not have to be updated or redeployed, it will automatically detect the closer region and will auto-home on to it should a regional event occur. In this case, when there is a West Europe region outage, the application will automatically start sending read and write operations to the next closest region so, North Europe.


# Summary

In this article, we discussed different scenarios related to geo-replication and multiple-writes handling in the Azure Cosmos DB. As we can see there are many different possible scenarios this is why we have to carefully plan high availability and failover for our applications. I encourage you to read more in the official [Azure Cosmos DB documentation](https://docs.microsoft.com/en-us/azure/cosmos-db/distribute-data-globally).
