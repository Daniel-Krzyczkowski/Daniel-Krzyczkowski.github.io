---
title: "Azure AD B2C Series - Azure Application Insights integration"
excerpt: "In this article would like to present how to integrate Azure Application Insights with custom policies to discover errors."
---

<p align="center">
<img src="/images/devisland/article24/assets/B2cSeriesAzureAppInsights1.png?raw=true" alt="Azure AD B2C Series - Azure Application Insights integration"/>
</p>

I had a chance to work with the Azure Active Directory B2C quite a lot recently and decided that it would be nice to share some knowledge about it. Just to make life easier for people using it especially when there are some custom usage scenarios. This is the the third article from the series and in this article I would like to present how to connect Azure Application Insights to the custom policies to detect errors in the Azure AD B2C Custom Policies (Identity Experience Framework).

There are also links to the great content after opening "Identity Experience Framework" tab in the Azure portal:

<p align="center">
<img src="/images/devisland/article24/assets/B2cSeriesAzureAppInsights2.PNG?raw=true" alt="Image not found"/>
</p>


## Introduction

In my two previous articles I presented [how to use custom claims (attributes) in the Identity Experience Framework](https://daniel-krzyczkowski.github.io/Azure-AD-B2C-Series-Custom-Policies-With-Custom-Claims/) and [how to call external service (Azure Function) during the login and registration flow](https://daniel-krzyczkowski.github.io/Azure-AD-B2C-Series-External-Service-Call/).


## Create Azure Application Insights instance

Before we move forward with custom policies cponfiguration, we have to create Azure Application Insights instance in the Azure portal. Once it is created, copy "instrumentation key" - we will use it later in the article:

<p align="center">
<img src="/images/devisland/article24/assets/B2cSeriesAzureAppInsights3.PNG?raw=true" alt="Image not found"/>
</p>


## Enable errors logging in the custom policies

Download "signup_signin" policy file from the Azure portal:

<p align="center">
<img src="/images/devisland/article24/assets/B2cSeriesAzureAppInsights4.PNG?raw=true" alt="Image not found"/>
</p>

We have to enable logging with Azure Application Insights. Update policy file with below code.


Update "TrustFrameworkPolicy" section with "DeploymentMode" set to "Development" and "UserJourneyRecorderEndpoint" set to "urn:journeyrecorder:applicationinsights":

```csharp
<TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  ....
  DeploymentMode="Development"
  UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights">
```

Next just right under the "DefaultUserJourney" element add "UserJourneyBehaviors" element with below code:

```csharp
    <UserJourneyBehaviors>
      <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="<<INSTRUMENTATION KEY>>" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
    </UserJourneyBehaviors>
```

Please note that in this part we have to provide Instrumentation Key we copied from the Azure portal.

## Verify logs in the Azure Application Insights

Try to sign in one or two times. After few minutes in the Azure Application Insights you should see the logs:

<p align="center">
<img src="/images/devisland/article24/assets/B2cSeriesAzureAppInsights5.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article24/assets/B2cSeriesAzureAppInsights6.PNG?raw=true" alt="Image not found"/>
</p>

You can verify the logs and check why the login or registration flow is not working properly.

<p align="center">
<img src="/images/devisland/article24/assets/B2cSeriesAzureAppInsights6.PNG?raw=true" alt="Image not found"/>
</p>

Of course you can enable logging for other policies too, for instance password reset or user profile update.


# Summary

In this article I presented how to call enable logging in the Azure AD B2C custom policies using Azure Application Insights. This is the last article from the series. I hope you found it interesting and helpful.