---
title: "Lost in Azure cloud identity - part 6"
excerpt: "This article presents how to use Microsoft Graph API to manage users in the Azure AD B2C"
header:
  image: /images/devisland/article70/assets/IdentityOnAzure-part6-1.png
---

<p align="center">
<img src="/images/devisland/article70/assets/IdentityOnAzure-part6-1.png?raw=true" alt="Lost in Azure cloud identity - part 6"/>
</p>


# Introduction

I have decided to create series related to identity and access management using Azure cloud services. Important note first - I will focus more on the development side and integration aspects. This series is focused on developers who would like to understand different concepts and mechanisms around identity using Azure cloud services like Azure Active Directory and Azure Active Directory B2C. It does not mean that if you are an architect or administrator, you will not find anything interesting. I think that this series can be helpful for everyone who wants to learn more about identity services in the Microsoft Azure cloud.

This is the sixth article from the series. In this article, we will talk about user management in the Azure AD B2C using Microsoft Graph API. This can be helpful if you need to migrate users from the external identity providers to the Azure Active Directory B2C but also to export user accounts to the external systems.

**Important**

*This series assumes that you already have some basic knowledge around identity concepts, application implementation, and Azure cloud services - at least Azure Active Directory.*

**Source code**

Source code of all applications and AD B2C custom policies are available on my [GitHub](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity)

**Links to additional helpful resources**

In this specific article, I will focus on custom policies. There is really good documentation I recommend you to check:

[Azure AD B2C user management with Microsoft Graph](https://docs.microsoft.com/en-us/azure/active-directory-b2c/microsoft-graph-operations#user-management)


**Solution architecture discussed in this series**

<p align="center">
<img src="/images/devisland/article70/assets/IdentityOnAzure-part6-2.png?raw=true" alt="Image not found"/>
</p>

The project I am going to discuss in this article is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/tree/main/src/tmf-identity-web-api). Tech Mind Factory Identity Web API is written using ASP .NET Core (.NET 5).

I used three NuGet packages in the project:

* [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph) - to call Microsoft Graph API using .NET C# SDK
* [Microsoft.Graph.Auth](https://www.nuget.org/packages/Microsoft.Graph.Auth/) - to authenticate with Microsoft Graph using client credentials flow (client id and client secret)
* [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web) - to secure access to the Tech Mind Factory Identity API, calling it without a valid access token will fail

Remember to check my previous articles from this series to learn more.


# Secure Web API with Azure AD B2C

First, it is important to clarify that Tech Mind Factory Identity Web API is secured by Azure AD B2C itself. It means that anonymous access is forbidden. You can read more about how to secure ASP .NET Core Web API with Azure AD B2C using Microsoft Identity Web library in [my second article](https://daniel-krzyczkowski.github.io/Lost-In-Azure-Cloud-Identity-Serie-Part2/) from this series.

In the *[Startup.cs](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/5a498b982a380c56ec38bc6c4da57d0d654253cc/src/tmf-identity-web-api/TMF.Identity.API/Startup.cs#L30)* file you can see that I use *AddMicrosoftIdentityWebApiAuthentication* method provided by the Microsoft Identity Web library.

The *[UserController](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/5a498b982a380c56ec38bc6c4da57d0d654253cc/src/tmf-identity-web-api/TMF.Identity.API/Controllers/UserController.cs#L11)* class has *Authorize* attribute added. This prevents accessing it without a valid access token.

In the *[appsettings.json](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/main/src/tmf-identity-web-api/TMF.Identity.API/appsettings.json)* file you can see that I added *[AzureAdB2C](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/5a498b982a380c56ec38bc6c4da57d0d654253cc/src/tmf-identity-web-api/TMF.Identity.API/appsettings.json#L13)* section where I provided configuration details.

Here is my Tech Mind Factory Identity Web API registered in the Azure AD B2C:

<p align="center">
<img src="/images/devisland/article70/assets/IdentityOnAzure-part6-3.PNG?raw=true" alt="Image not found"/>
</p>

## Grant API permission to read and write all users' full profiles

We want to grant permission to this Web API application to manage user accounts in the Azure AD B2C instance. To do this we need to grant *User.ReadWrite.All* application permission using below steps:

<p align="center">
<img src="/images/devisland/article70/assets/IdentityOnAzure-part6-6.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article70/assets/IdentityOnAzure-part6-5.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article70/assets/IdentityOnAzure-part6-7.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article70/assets/IdentityOnAzure-part6-8.PNG?raw=true" alt="Image not found"/>
</p>

Once we grant above permission (remember to click *Grant admin consent for Tech Mind Factory*), we can use Microsoft Graph API to add, update, delete, and read user profiles in our Azure AD B2C tenant:

<p align="center">
<img src="/images/devisland/article70/assets/IdentityOnAzure-part6-4.PNG?raw=true" alt="Image not found"/>
</p>

## Generate application secret

To obtian access token using client credentials flow we have to also generate client secret for our application (we can also use certificates but to simplify the process I used secret). I generated the secret for the TMF Identity app registered in the Azure AD B2C:

<p align="center">
<img src="/images/devisland/article70/assets/IdentityOnAzure-part6-9.PNG?raw=true" alt="Image not found"/>
</p>


# Using Microsoft Graph SDK

Once we add [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph) and [Microsoft.Graph.Auth](https://www.nuget.org/packages/Microsoft.Graph.Auth/) NuGet packages I mentioned at the beginning of this article, we are ready to use Microsoft Graph SDK in the TMF Identity Web API project.


## Microsoft Graph SDK configuration

To use Microsoft Graph API, we have to first authenticate and get an access token. First, we have to add configuration parameters in the *[appsettings.json](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/main/src/tmf-identity-web-api/TMF.Identity.API/appsettings.json#L19)* file. We need four parameters described below:

* TenantId - ID of our AD B2C tenant. You can find this information in the Azure AD Overview tab of your AD B2C tenant
* AppId - client ID value of the registered API application described above
* AppSecret - secret value we generated for the API application above
* TenantName - the name of our AD B2C tenant with *.onmicrosoft.com* suffix


## Microsoft Graph SDK GraphServiceClient instance registration

Microsoft Graph SDK provides *GraphServiceClient* instance which can be use to send requests and get responses. In the *[UserManagementServicesCollectionExtensions](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/main/src/tmf-identity-web-api/TMF.Identity.API/Core/DependencyInjection/UserManagementServicesCollectionExtensions.cs)* class you can see that I register *GraphServiceClient* instance as singleton:

 ```csharp
            services.AddSingleton<IGraphServiceClient>(implementationFactory =>
            {
                var msGraphServiceConfiguration = implementationFactory.GetRequiredService<IMsGraphServiceConfiguration>();
                IConfidentialClientApplication confidentialClientApplication = ConfidentialClientApplicationBuilder
                    .Create(msGraphServiceConfiguration.AppId)
                    .WithTenantId(msGraphServiceConfiguration.TenantId)
                    .WithClientSecret(msGraphServiceConfiguration.AppSecret)
                    .Build();

                ClientCredentialProvider authProvider = new ClientCredentialProvider(confidentialClientApplication);
                return new GraphServiceClient(authProvider);
            });

            services.AddSingleton<IUserManagementService, MsGraphUserManagementService>();
```

As you can see above, I pass *ClientCredentialProvider* instance as a constructor parameter for the *GraphServiceClient*. This *ClientCredentialProvider* uses parameters from the *appsettings.json*, from the *MicrosoftGraph* section.

You can also see that I register *[MsGraphUserManagementService](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/main/src/tmf-identity-web-api/TMF.Identity.Infrastructure/Services/MsGraphUserManagementService.cs)* instance - this is the class I created to wrap *GraphServiceClient* and its operations. Below you can see the example of a method to retrieve information about the specific user by ID:

 ```csharp
        public async Task<UserEntity> GetUserAsync(string userId)
        {
            try
            {
                var user = await _graphServiceClient.Users[userId]
                                 .Request()
                                 .Select(e => new
                                 {
                                     e.Id,
                                     e.GivenName,
                                     e.Surname,
                                     e.Identities
                                 })
                                 .GetAsync();

                var email = user.Identities.ToList()
                            .FirstOrDefault(i => i.SignInType == "emailAddress")
                            ?.IssuerAssignedId;

                return new UserEntity
                {
                    Id = user.Id,
                    FirstName = user.GivenName,
                    LastName = user.Surname,
                    Email = email
                };
            }

            catch (ServiceException ex)
            {
                if (ex.StatusCode == System.Net.HttpStatusCode.NotFound)
                {
                    return null;
                }

                else
                {
                    throw;
                }
            }
        }
```


There is also operation for retrieving all users from the Azure AD B2C:


 ```csharp
        public async Task<IReadOnlyList<UserEntity>> GetAllUsersAsync()
        {
            List<User> users = new List<User>();
            try
            {

                IGraphServiceUsersCollectionPage iGraphServiceUsersCollectionPage = await _graphServiceClient.Users
                                                                       .Request()
                                                                       .Select($"id," +
                                                                       $" userPrincipalName," +
                                                                       $" givenName," +
                                                                       $" surname")
                                                                       .Top(50)
                                                                       .GetAsync();

                var userPageIterator = PageIterator<User>.CreatePageIterator(_graphServiceClient,
                                                           iGraphServiceUsersCollectionPage,
                                                           entity => { users.Add(entity); return true; });

                await userPageIterator.IterateAsync();
            }

            catch (ServiceException ex)
            {
                if (ex.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
                {
                    _logger.LogError(LoggerEvents.MicrosoftGraphTooManyRequests, string.Concat(ex.Message, " ", ex.InnerException?.Message));
                    var retryAfter = ex.ResponseHeaders.RetryAfter.Delta.Value;
                    await Task.Delay(TimeSpan.FromSeconds(retryAfter.TotalSeconds));
                    await GetAllUsersAsync();
                }

                else
                {
                    throw;
                }
            }

            var userEntities = users
                                  .Select(u => new UserEntity()
                                  {
                                      Id = u.Id,
                                      Email = u.UserPrincipalName,
                                      FirstName = u.GivenName,
                                      LastName = u.Surname
                                  })
                                  .ToList();

            return userEntities;
        }
```

Please note that in the above method I use *PageIterator* which enables asynchronous iteration. You can also see that in the *catch* block I check if *HttpStatusCode.TooManyRequests* was returned. If yes, we have to wait for the amount of time provided in the *RetryAfter* header returned from the Microsoft Graph API. It is good to be aware that Microsoft Graph implements a throttling pattern so in case of too many requests we have to wait a specific amount of time to call the API again with success.

I encourage you to check the full source code on [my GitHub](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/tree/main/src/tmf-identity-web-api) to see how to integrate with Microsoft Graph to manage users in the Azure AD B2C tenant.


# Summary

In this article, we talked about Microsoft Graph SDK integration in the ASP .NET Core Web API project. Now you know how to use Microsoft Graph API to manage users in the Azure AD B2C tenant. This kind of knowledge can be helpful when you have to migrate users from other identity systems to the Azure Active Directory B2C.
