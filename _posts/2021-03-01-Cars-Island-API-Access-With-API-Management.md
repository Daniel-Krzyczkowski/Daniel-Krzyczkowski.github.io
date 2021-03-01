---
title: "Controlled access to Cars Island ASP .NET Core API with Azure API Management - part 9"
excerpt: "This article presents how to use Azure API Management to control access to APIs endpoints"
header:
  image: /images/devisland/article58/assets/CarsIslandAzureApiManagement1.jpg
---

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement1.jpg?raw=true" alt="Controlled access to Cars Island ASP .NET Core API with Azure API Management - part 9"/>
</p>


# Introduction

In [my first article](https://daniel-krzyczkowski.github.io/Cars-Island-Car-Rental-On-Azure-Cloud/), I introduced you to Cars Island car rental on the Azure cloud. I created this fake project to present how to use different Microsoft Azure cloud services and how to their SDKs. I also presented what will be covered in the next articles. Here is the next article from the series where I would like to discuss how to use Azure API Management to secure access to internal APIs. In this specific scenario, we are going to discuss how Cars Island ASP. NET Core API is hidden behind Azure API Management and what policies are used to secure it.

In the Cars Island solution, Azure API Management is used to control and secure access to the Cars Island ASP .NET Core API. Below I present solution architecture:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureFunctionsWithGraphApi2.png?raw=true" alt="Image not found"/>
</p>


*[Cars Island project is available on my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure)*


# Azure API Management structure and capabilities

Azure API Management is a service to create consistent and modern API gateways for existing back-end services. It provides secure, scalable API access for your applications. Azure API Management consists of three main components:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement4.PNG?raw=true" alt="Image not found"/>
</p>

## API Gateway Capabilities

* Accepts API calls and routes them to your backends
* Verifies API keys, JWT tokens, certificates, and other credentials
* Enforces usage quotas and rate limits
* Caches backend responses

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement3.PNG?raw=true" alt="Image not found"/>
</p>


## Azure Portal Capabilities

* Define or import API schema
* Set up policies like quotas or transformations on the APIs
* Package APIs into products
* Manage users


## Developer Portal Capabilities

* Read the API documentation
* Create an account and subscribe to get API keys
* Try out an API via the interactive console
* Access analytics

## Versions and Revisions

### Versions

Versions allow presenting of groups of related APIs to the developers. You can use versions to handle breaking changes in your API safely:

```xml
    https://apis.cars-island.com/car/all/v1
    https://apis.cars-island.com/car/all/v2
```


### Revisions
Revisions allow you to make changes to the APIs in a controlled and safe way, without disturbing your API consumers:


```xml
    https://apis.cars-island.com/car/all;rev=3
```

*Typically versions are used to separate API versions with breaking changes, while revisions can be used for minor and non-breaking changes to an API*


## Products and Groups

In the Azure API Management, there is also a concept of products and groups. You can think about the products like about suite of APIs that is available to consume. Group then is a target audience that can consume this suite of APIs:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement5.PNG?raw=true" alt="Image not found"/>
</p>

There are three initial groups available in the Azure API Management but you can extend them:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement6.PNG?raw=true" alt="Image not found"/>
</p>


Products can be *Open* or *Protected*. Protected products must be subscribed to before they can be used. Subscription approval is configured at the product level. Developers need this subscription to access products. Developers can be created or invited to join by administrators, or they can sign up from the Developer portal. Each developer is a member of one or more groups, and can subscribe to the products that grant visibility to those groups.

*Administrators can also create custom groups or leverage external groups in associated Azure Active Directory tenants*


## Policies

Policies are a powerful capability of Azure API Management that allows changing the behavior of the API through configuration. Policies are a collection of statements that are executed sequentially on the request or response of an API.

Example of policies:

* Format conversion from XML to JSON
* Restrict the number of incoming calls
* Caches response according to the specified cache-control configuration

There are different categories of policies in the Azure API Management:


### Access Restriction Policies

* Validate JWT tokens - enforces existence and validity of a JWT token in header or query parameter
* Check HTTP header presence - enforces existence and/or value of a HTTP header
* Limit call rate by subscription - prevents API usage by limiting call rate, on a per subscription basis


### Advanced Policies

* Mock response - returns a mocked response directly to the caller
* Retry - retries execution of a request at the specified time intervals
* Forward request - forwards the request to the backend service


### Transformation Policies

* Convert XML to JSON - converts request or response body from XML to JSON
* Convert JSON to XML - converts request or response body from JSON to XML
* Find and replace string in body - finds a request or response substring and replaces it with a different substring


### Caching Policies

* Store to cache - caches response according to the specified cache control configuration
* Get from cache - perform cache look up and return a valid cached response when available
* Remove value from cache - remove an item in the cache by key


There are many more policies available in the Azure API Management.

### Policy scopes

Policies can have different scopes. There are four scopes available:

* Global scope - affects all APIs within the instance of API Management
* Product scope - manages access to the product as a single entity
* API scope - affects only a single API
* Operation scope - affects only one operation within the API

There can be also a question when do policies execute?

* **Inbound** policies execute when a request is received from a client
* **Backend** policies execute before a request is forwarded to a managed API
* **Outbound** policies execute before a response is sent to a client
* **On-Error** policies execute when an exception is raised


### Policy structure example

Below you can see the structure of sample policy from the Azure API Management. Pleae note that there is quota limit set for the requests:

```xml
<policies>
    <inbound>
        <rate-limit calls="5" renewal-period="10" />
        <base />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```


# Azure API Management in the Cars Island project

In the Cars Island project API Management was used to provide access to the Cars Island ASP .NET Core Web API. I wanted to hide the API behind the API Management and restrict the access. Below you can see the Cars Island API Management's Developer Portal I configured:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement7.PNG?raw=true" alt="Image not found"/>
</p>

Developers have to register first and once they are approved by the Administrator, they can start using Cars Island API.

## Import an API

The first important step is to import API definition to the Azure API Management. Cars Island has its Open API definition which can be accessed using the endpoint:

```xml
    https://app-cars-island-api-dev.azurewebsites.net/swagger/v1/swagger.json
```

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement9.PNG?raw=true" alt="Image not found"/>
</p>

We can import API using *Add new API* tab and selecting *Open API*:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement10.PNG?raw=true" alt="Image not found"/>
</p>

In the dialog, we have to provide above URL to Open API definition. Rest of the fields will be filled out automatically:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement11.PNG?raw=true" alt="Image not found"/>
</p>

Once we click *Create* button, API should become visible on the list:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement12.PNG?raw=true" alt="Image not found"/>
</p>

We have to then provide *Web service URL* under the *Settings* tab of our API:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement18.PNG?raw=true" alt="Image not found"/>
</p>

We can then test endpoints using *Test* tab.


## Setup product and assign API

Using *Products* tab, we can either use existing products (*Starter* or *Unlimited*), or we can create one. With *Starter* product, subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week. This can be ideal in scenarios when we want to provide trial access to our APIs.

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement13.PNG?raw=true" alt="Image not found"/>
</p>

I have assigned Cars Island API to the *Unlimited* product, because I want to provide full access to developers:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement14.PNG?raw=true" alt="Image not found"/>
</p>

You can ask then how I can verify who is using this API? In the Settings section there is a checkbox *Requires approval*. It means that even if developer will sign up, there will be approval from Administrator required:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement15.PNG?raw=true" alt="Image not found"/>
</p>

We can also decide whether products (with APIs) will be available for *Guests* or not using *Access control* tab:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement16.PNG?raw=true" alt="Image not found"/>
</p>


## Provide access to developers

Once we setup products and APIs, we can publish API Management developer portal and provide access for developers. Under the section called *Portal overview* there are two important buttons:

* Developer portal - with this button we can open the developer portal and customize its look & feel
* Publish - this button enables us to publish the developer portal so developers can access it

Once developer portal is published, we can access it:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement7.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement8.PNG?raw=true" alt="Image not found"/>
</p>


## Configure policies

We can also configure policies I described above to control the access to the APIs behind the API Management. [Here](https://docs.microsoft.com/en-us/azure/api-management/api-management-policies) is great documentation where you can find a different kind of policies. The below screenshot presents a list of policies that we can apply to one single endpoint in our API:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement18.PNG?raw=true" alt="Image not found"/>
</p>

In the Cars Island project I have used caching policy to cache all available cars, and JWT token validation policy when calling new car reservation endpoint:

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement22.PNG?raw=true" alt="Image not found"/>
</p>

```xml
<policies>
    <inbound>
        <cache-lookup vary-by-developer="false" vary-by-developer-groups="false"
                      caching-type="internal" />
        <base />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <cache-store duration="60" />
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement23.PNG?raw=true" alt="Image not found"/>
</p>

```xml
<policies>
    <inbound>
        <base />
        <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
            <openid-config url="https://carsisland.b2clogin.com/carsisland.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=B2C_1A_SignUpOrSignin" />
            <audiences>
                <audience>50f00e2d-xxx</audience>
            </audiences>
            <required-claims>
                <claim name="scp" match="all">
                    <value>access_as_user</value>
                </claim>
            </required-claims>
        </validate-jwt>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```


## Call APIs endpoints

Once everything is configured on the Azure API Management side, we can call the API using the subscription key obtained, after we registered as a developer (using sign up button from the developer portal). To call any endpoint we need to add *Ocp-Apim-Subscription-Key* HTTP header. Without this header, we will not be authorized to call any endpoint of our API.


<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement189.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article58/assets/CarsIslandAzureApiManagement20.PNG?raw=true" alt="Image not found"/>
</p>


# Summary

In this article, I described how to set up and use Azure API Management to control and secure access to APIs. Source code of the Cars Island solution is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure) so you can see all implementation details. I encourage you to watch my course on Pluralsight called [Microsoft Azure Developer: Implement API Management](https://www.pluralsight.com/courses/microsoft-azure-developer-implement-api-management) where I explained how to use Microsoft Azure API Management to control who uses your APIs, to enforce usage policies, and to present a professional front-end to developers using the API.

If you want to learn more about Azure API Management, you can also check this [official documentation](https://docs.microsoft.com/en-us/azure/api-management/). 
