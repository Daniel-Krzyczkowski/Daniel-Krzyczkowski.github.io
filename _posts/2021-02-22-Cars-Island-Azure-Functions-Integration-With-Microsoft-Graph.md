---
title: "Cars Island Azure Functions - integration with Microsoft Graph SDK and Azure AD B2C - part 8"
excerpt: "This article presents how to implement the communication with Azure AD B2C using Microsoft Graph API"
header:
  image: /images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi1.jpg
---

<p align="center">
<img src="/images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi1.jpg?raw=true" alt="Cars Island Azure Functions - integration with Microsoft Graph SDK and Azure AD B2C - part 8"/>
</p>


# Introduction

In [my first article](https://daniel-krzyczkowski.github.io/Cars-Island-Car-Rental-On-Azure-Cloud/), I introduced you to Cars Island car rental on the Azure cloud. I created this fake project to present how to use different Microsoft Azure cloud services and how to their SDKs. I also presented what will be covered in the next articles. Here is the next article from the series where I would like to discuss how to communicate with Azure AD B2C using Microsoft Graph SDK in the Azure Function App project.

In the Cars Island solution, Azure AD B2C service is used to authenticate users. In the Azure AD B2C there is user profile data stored. Below I present solution architecture:

<p align="center">
<img src="/images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi2.png?raw=true" alt="Image not found"/>
</p>

Once the new reservation is created, a new message is added to the Azure Service Bus Queue. Then, Mail Notifications Event Handler Function App is triggered to send a new email to the customer. To get data about customers (email, first name, and last name), Azure Function has to communicate with Azure AD B2C using Microsoft Graph SDK.


*[Cars Island project is available on my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure)*


# Application registration in the Azure AD B2C

To cumminicate with the Azure AD B2C using Microsoft Graph, we must register the application with specific permissions in the Azure AD B2C portal:

<p align="center">
<img src="/images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi7.PNG?raw=true" alt="Image not found"/>
</p>

Once we register new application, it has unique *Client ID* value assigned. We will need this value to obtain access token to call the Microsoft Graph API:

<p align="center">
<img src="/images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi6.PNG?raw=true" alt="Image not found"/>
</p>

We have to also generate new app secret which is also required to obtain access token to call the Microsoft Graph API. It can be created under *Certificates & secrets* section:

<p align="center">
<img src="/images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi5.PNG?raw=true" alt="Image not found"/>
</p>

Last step is to grant our newly registered permission to read basic profiles of the users that are registered in the Azure AD B2C. To do it, select *Api permissions* from the left bar and select *Microsoft Graph API*:

<p align="center">
<img src="/images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi3.PNG?raw=true" alt="Image not found"/>
</p>

Then select  *Application permissions* and type *user.read* in the search box. You should select below *User.Read.All* permission:

<p align="center">
<img src="/images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi4.PNG?raw=true" alt="Image not found"/>
</p>

The last step required, we have to grant admin consent for specified permissions:

<p align="center">
<img src="/images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi8.PNG?raw=true" alt="Image not found"/>
</p>

Now we are ready to integrate with Microsoft Graph in the source code of the Azure Function App.


# Microsoft Graph .NET SDK integration

Integration with Microsoft Graph is really easy using dedicated NuGet packages. In the [*CarsIsland.MailSender.Infrastructure*](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/tree/master/src/func-app/CarsIsland.MailSender.Infrastructure) you can see that below two packages are used:

* [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph) - this package enabled easy communication with Microsoft Graph API
* [Microsoft.Graph.Auth](https://www.nuget.org/packages/Microsoft.Graph.Auth/) - this package makes it easy to obtain access tokens to call Microsoft Graph API

First, we have to register *GraphServiceClient* instance in the DI container. It is done in the [*UserManagementServiceCollectionExtensions*](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/func-app/CarsIsland.MailSender.FuncApp/Core/DependencyInjection/UserManagementServiceCollectionExtensions.cs) class:

```csharp
        public static IServiceCollection AddUserManagementServices(this IServiceCollection services)
        {
            services.AddScoped<IGraphServiceClient>(implementationFactory =>
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

            services.AddScoped<IMsGraphSdkClientService, MsGraphSdkClientService>();

            return services;
        }
```

As you can see, we use *IConfidentialClientApplication* provided by *Microsoft.Graph.Auth* NuGet package to obtain access tokens. Please note that above there are parameters passes that we got from the Azure portal: *AppId*, *AppSecret*, and *TenantId*. 

Once *GraphServiceClient* is registered we can call Microsoft Graph API with it. I created dedicated class called [*MsGraphSdkClientService *](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/func-app/CarsIsland.MailSender.Infrastructure/Services/MsGraph/MsGraphSdkClientService.cs) where *GraphServiceClient* instance is injected:

```csharp
    public class MsGraphSdkClientService : IMsGraphSdkClientService
    {
        private readonly IMsGraphServiceConfiguration _msGraphServiceConfiguration;
        private readonly IGraphServiceClient _graphServiceClient;

        public MsGraphSdkClientService(IMsGraphServiceConfiguration msGraphServiceConfiguration,
                                       IGraphServiceClient graphServiceClient)
        {
            _msGraphServiceConfiguration = msGraphServiceConfiguration
                 ?? throw new ArgumentNullException(nameof(msGraphServiceConfiguration));

            _graphServiceClient = graphServiceClient
                 ?? throw new ArgumentNullException(nameof(graphServiceClient));
        }

        public async Task<UserAccount> GetUserAsync(string userId)
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

            if (user != null)
            {
                var email = user.Identities.ToList()
                            .FirstOrDefault(i => i.SignInType == "emailAddress")
                            ?.IssuerAssignedId;

                return new UserAccount
                {
                    Id = user.Id,
                    FirstName = user.GivenName,
                    LastName = user.Surname,
                    Email = email
                };
            }

            else
            {
                return null;
            }
        }
    }
```

As you can see, in the *GetUserAsync* method there is a call to Microsoft Graph API to retrieve information about the basic profile of the user - we have to provide user ID (the unique identifier of the user assigned by the Azure AD B2C). We need the below information to include in the confirmation email:

* GivenName - the first name of the user
* Surname - last name of the user
* Identities - to get user email

Finally, in the [*CarReservationMessagingService*](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/func-app/CarsIsland.MailSender.Infrastructure/Services/Integration/CarReservationMessagingService.cs) class we use above *MsGraphSdkClientService* to get information about the customer, before we can send email confirmation:

```csharp
    var customer = await _msGraphSdkClientService.GetUserAsync(carReservationIntegrationMessage.CustomerId);
```

Final result is presetned below, in the email message we include customer first name and last name retrieved from the Azure AD B2C:

<p align="center">
<img src="/images/devisland/article57/assets/CarsIslandAzureFunctionsWithGraphApi9.png?raw=true" alt="Image not found"/>
</p>


# Summary

In this article, I described how to get information about the users registered in the Azure AD B2C using Microsoft Graph. With the Microsoft Graph NuGet package, we can integrate with Microsoft Graph in the Azure Function App project. Source code of the Cars Island solution is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure) so you can see all implementation details.

If you want to learn more about managing users in the Azure AD B2C using Microsoft Graph API, you can also check this [official documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/microsoft-graph-operations). 

