---
title: "Cars Island ASP .NET Core API - integration with Azure Storage Account - part 4"
excerpt: "This article presents how to integrate ASP .NET CORE API with Azure Blob Storage using the new .NET C# library."
header:
  image: /images/devisland/article53/assets/CarsIslandWebApiAzureStorageAccount1.jpg
---

<p align="center">
<img src="/images/devisland/article53/assets/CarsIslandWebApiAzureStorageAccount1.jpg?raw=true" alt="Cars Island ASP .NET Core API - integration with Azure Storage Account - part 4"/>
</p>


# Introduction

In [my first article](https://daniel-krzyczkowski.github.io/Cars-Island-Car-Rental-On-Azure-Cloud/), I introduced you to Cars Island car rental on the Azure cloud. I created this fake project to present how to use different Microsoft Azure cloud services and how to their SDKs. I also presented what will be covered in the next articles. Here is the fourth article from the series where I would like to discuss how to integrate with Azure Storage Account and use the new .NET C# library.

In the Cars Island solution, Azure Storage Account is used to store cars images, and files attached to the customer's enquiries. Below I present solution architecture:

<p align="center">
<img src="/images/devisland/article53/assets/CarsIslandWebApiAzureStorageAccount2.png?raw=true" alt="Image not found"/>
</p>

Car images are publicly accessible so anybody with the link can display car images in the browser. Files attached to the enquiries are private.

*[Cars Island project is available on my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure)*


<p align="center">
<img src="/images/devisland/article53/assets/CarsIslandWebApiAzureStorageAccount3.PNG?raw=true" alt="Image not found"/>
</p>


# Azure Storage Account setup

## Azure Storage Account creation

First of all, we have to create new instance of Azure Storage Account and configure it. I created the service with below configuration:

<p align="center">
<img src="/images/devisland/article53/assets/CarsIslandWebApiAzureStorageAccount4.PNG?raw=true" alt="Image not found"/>
</p>

As you can see I selected *General-purpose v2 account*. This kind of Storage Account supports below Azure Storage services:

- Blobs (all types: Block, Append, Page)
- Data Lake Gen2
- Files
- Disks
- Queues
- Tables

Microsoft recommends general-purpose v2 accounts for most scenarios. For the *Replication* I selected *Read-access geo-redundant storage (RA-GRS)*. Let's stop and discuss Azure Storage Account data redundancy.


## Azure Storage Account - data redundancy and high availability

When you create an Azure Storage Account it is worth considering the tradeoffs between lower costs and higher availability. You have to decide what kind of data you plan to store and how important this data is to make your solution high available.

Azure Storage provides below redundancy options to choose:

**Locally redundant storage (LRS)**

A simple, low-cost redundancy strategy. Data is copied synchronously three times within a single physical location in the primary region. Locally redundant storage protects your data against server rack and drive failures. However, if a disaster such as fire or flooding occurs within the data center, all replicas of a storage account using LRS may be lost or unrecoverable. This is why it is better to choose one of the options described below.

**Zone-redundant storage (ZRS)** 

Redundancy for scenarios requiring high availability. Zone-redundant storage (ZRS) replicates your Azure Storage data synchronously across three Azure availability zones in the primary region. For protection against regional disasters, Microsoft recommends using geo-zone-redundant storage (GZRS), which uses ZRS in the primary region and also geo-replicates your data to a secondary region.

**Geo-redundant storage (GRS)** 

Cross-regional redundancy to protect against regional outages. Data is copied synchronously three times in the primary region, then copied asynchronously to the secondary region. However, that data is available to be read-only if the customer or Microsoft initiates a failover from the primary to secondary region. For read access to data in the secondary region, you have to enable read-access geo-redundant storage (RA-GRS). When you enable read access to the secondary region, your data is available to be read at all time.

**Geo-zone-redundant storage (GZRS)**

Redundancy for scenarios requiring both high availability and maximum durability. Data is copied synchronously across three Azure availability zones in the primary region, then copied asynchronously to the secondary region. For read access to data in the secondary region, enable read-access geo-zone-redundant storage (RA-GZRS).


I also encourage you to watch my course on Pluralsight called [Designing a Disaster Recovery Strategy on Microsoft Azure](https://www.pluralsight.com/courses/microsoft-azure-disaster-recovery-strategy-designing-update) where I explained what are the possible options in the Azure Storage Account related to geo-replication and high availability.

As I mentioned above, in the Cars Island solution there is *Read-access Geo-redundant Storage Account* replication used:


<p align="center">
<img src="/images/devisland/article53/assets/CarsIslandWebApiAzureStorageAccount5.png?raw=true" alt="Image not found"/>
</p>


# Azure Storage Account .NET SDK integration

In the Cars Island project, I integrated the ASP .NET Core Web API application with Azure Storage Account using [new library](https://www.nuget.org/packages/Azure.Storage.Blobs).  I decided to use this library because it follows best practices and implement common patterns. If you want to learn more about new Azure SDKs, please check this [official documentation](https://azure.microsoft.com/en-us/downloads/).

Let's start with an explanation of how the initial setup looks like. In the *[StorageServiceCollectionExtensions.cs](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.API/Core/DependencyInjection/StorageServiceCollectionExtensions.cs)* file you can find out how *BlobServiceClient* is instantiated. To follow [best practices related to lifetime management](https://devblogs.microsoft.com/azure-sdk/lifetime-management-and-thread-safety-guarantees-of-azure-sdk-net-clients/), *BlobServiceClient* type is registered as a singleton in the IoC container. You can also ask why I have used *services.AddSingleton* with *implementationFactory* instead of just using *services.AddSingleton*. I did it because it enables automatic object disposal so once the application is closed, all resources will be released automatically. You can read more about [service registration methods](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#service-registration-methods).


```csharp
    public static class StorageServiceCollectionExtensions
    {
        public static IServiceCollection AddStorageServices(this IServiceCollection services)
        {
            var serviceProvider = services.BuildServiceProvider();

            var storageConfiguration = serviceProvider.GetRequiredService<IBlobStorageServiceConfiguration>();

            services.AddSingleton(factory => new BlobServiceClient(storageConfiguration.ConnectionString));
            services.AddSingleton<IBlobStorageService, BlobStorageService>();
            return services;
        }
    }
```

# Data operations

Once we discussed Azure Storage Account setup and integration with the .NET SDK in the source code, we can discuss how it can be used.

There is [*BlobStorageService*](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.Infrastructure/Services/Storage/BlobStorageService.cs) instance registered. Let's discuss its structure. As you can see below *BlobStorageService* class uses *BlobServiceClient* instance to communicate with the Azure Storage Account. There are methods for deleting the blob, checking if blob exists or even operation responsible for generating shared access signatures (SAS):

```csharp
    public class BlobStorageService : IBlobStorageService
    {
        private readonly IBlobStorageServiceConfiguration _blobStorageServiceConfiguration;
        private readonly BlobServiceClient _blobServiceClient;

        ...

        public async Task DeleteBlobIfExistsAsync(string blobName)
        {
            try
            {
                var container = await GetBlobContainer();
                var blockBlob = container.GetBlobClient(blobName);
                await blockBlob.DeleteIfExistsAsync();
            }
            catch (RequestFailedException ex)
            {
                Log.Error($"Document {blobName} was not deleted successfully - error details: {ex.Message}");
                throw;
            }
        }

        public async Task<bool> DoesBlobExistAsync(string blobName)
        {
            try
            {
                var container = await GetBlobContainer();
                var blockBlob = container.GetBlobClient(blobName);
                var doesBlobExist = await blockBlob.ExistsAsync();
                return doesBlobExist.Value;
            }
            catch (RequestFailedException ex)
            {
                Log.Error($"Document {blobName} existence cannot be verified - error details: {ex.Message}");
                throw;
            }
        }

        public async Task DownloadBlobIfExistsAsync(Stream stream, string blobName)
        {
            try
            {
                var container = await GetBlobContainer();
                var blockBlob = container.GetBlobClient(blobName);

                await blockBlob.DownloadToAsync(stream);

            }

            catch (RequestFailedException ex)
            {
                Log.Error($"Cannot download document {blobName} - error details: {ex.Message}");
                if (ex.ErrorCode != "404")
                {
                    throw;
                }
            }
        }

        public async Task<string> GetBlobUrl(string blobName)
        {
            try
            {
                var container = await GetBlobContainer();
                var blob = container.GetBlobClient(blobName);

                string blobUrl = blob.Uri.AbsoluteUri;
                return blobUrl;
            }
            catch (RequestFailedException ex)
            {
                Log.Error($"Url for document {blobName} was not found - error details: {ex.Message}");
                throw;
            }
        }

        public async Task<string> UploadBlobAsync(Stream stream, string blobName)
        {
            try
            {
                Debug.Assert(stream.CanSeek);
                stream.Seek(0, SeekOrigin.Begin);
                var container = await GetBlobContainer();

                BlobClient blob = container.GetBlobClient(blobName);
                await blob.UploadAsync(stream);
                return blob.Uri.AbsoluteUri;
            }

            catch (RequestFailedException ex)
            {
                Log.Error($"Document {blobName} was not uploaded successfully - error details: {ex.Message}");
                throw;
            }
        }

        private async Task<BlobContainerClient> GetBlobContainer()
        {
            try
            {
                BlobContainerClient container = _blobServiceClient
                                .GetBlobContainerClient(_blobStorageServiceConfiguration.ContainerName);

                await container.CreateIfNotExistsAsync();

                return container;
            }
            catch (RequestFailedException ex)
            {
                Log.Error($"Cannot find blob container: {_blobStorageServiceConfiguration.ContainerName} - error details: {ex.Message}");
                throw;
            }
        }

        public string GenerateSasTokenForContainer()
        {
            BlobSasBuilder builder = new BlobSasBuilder();
            builder.BlobContainerName = _blobStorageServiceConfiguration.ContainerName;
            builder.ContentType = "video/mp4";
            builder.SetPermissions(BlobAccountSasPermissions.Read);
            builder.StartsOn = DateTimeOffset.UtcNow;
            builder.ExpiresOn = DateTimeOffset.UtcNow.AddMinutes(90);
            var sasToken = builder
                        .ToSasQueryParameters(new StorageSharedKeyCredential(_blobStorageServiceConfiguration.AccountName,
                                                                             _blobStorageServiceConfiguration.Key))
                        .ToString();
            return sasToken;
        }
    }
```

*BlobStorageService* is used in the *[EnquiryController](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/web-api/CarsIsland.API/Controllers/EnquiryController.cs)*. Once there is file attached to the new customer's enquiry, it is stored on the blob storage in the Azure cloud.

# Summary

In this article, I described how to set up an Azure Storage Account and how to integrate it into the ASP .NET Core Web API application. Source code of the Cars Island solution is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure) so you can see all implementation details.

If you want to learn more about the Azure Storage Account service, check this [official documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction).
