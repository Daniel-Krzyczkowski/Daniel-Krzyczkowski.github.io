---
title: "Lost in Azure cloud identity - part 8"
excerpt: "This article presents how to monitor Azure AD B2C with Azure Monitor"
header:
  image: /images/devisland/article72/assets/IdentityOnAzure-part8-1.png
---

<p align="center">
<img src="/images/devisland/article72/assets/IdentityOnAzure-part8-1.png?raw=true" alt="Lost in Azure cloud identity - part 8"/>
</p>


# Introduction

I have decided to create series related to identity and access management using Azure cloud services. Important note first - I will focus more on the development side and integration aspects. This series is focused on developers who would like to understand different concepts and mechanisms around identity using Azure cloud services like Azure Active Directory and Azure Active Directory B2C. It does not mean that if you are an architect or administrator, you will not find anything interesting. I think that this series can be helpful for everyone who wants to learn more about identity services in the Microsoft Azure cloud.

This is the last article from the series where I would like to write a little bit about integration with Azure Monitor to route Azure Active Directory B2C sign-in and auditing logs to different monitoring solutions. In this specific article, we will see how to enable log routing to the Log Analytics Workspace.

<p align="center">
<img src="/images/devisland/article72/assets/IdentityOnAzure-part8-3.png?raw=true" alt="Image not found"/>
</p>

**Important**

*This series assumes that you already have some basic knowledge around identity concepts, application implementation, and Azure cloud services - at least Azure Active Directory.*

**Source code**

Source code of all applications and AD B2C custom policies are available on my [GitHub](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity)

**Links to additional helpful resources**

In this specific article, I will focus on the final result of the integration. If you want to setup integration between your Azure AD B2C tenant and Azure Monitor, follow steps in the official documentation:

[Monitor Azure AD B2C with Azure Monitor](https://docs.microsoft.com/en-us/azure/active-directory-b2c/azure-monitor)


**Solution architecture discussed in this series**

<p align="center">
<img src="/images/devisland/article72/assets/IdentityOnAzure-part8-2.png?raw=true" alt="Image not found"/>
</p>



## Workbook for the audit logs data

Workbooks provide a flexible canvas for data analysis and the creation of rich visual reports within the Azure portal. I created workboob accordingly to the documentation and here is the final result:

<p align="center">
<img src="/images/devisland/article72/assets/IdentityOnAzure-part8-4.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article72/assets/IdentityOnAzure-part8-5.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article72/assets/IdentityOnAzure-part8-6.PNG?raw=true" alt="Image not found"/>
</p>


As you can see there are two tabs:
* *User Insights* - provides information about the platform (operating system), browser (which browser was used to authenticate), and country so you can see from which parts of the world your users come from
* *Authentications* - in this tab, you can check authentications per client (app registered in the Azure AD B2C), you can check custom policies that were used, or successful/failed login attempts


You can also switch to the *Logs* tab and use *Kusto* queries to display charts:

<p align="center">
<img src="/images/devisland/article72/assets/IdentityOnAzure-part8-7.PNG?raw=true" alt="Image not found"/>
</p>


## Alerts for specific events

Once the Log Analytics Workspace is ready and you have your workbook created, you can benefit from using *Alerts*:

<p align="center">
<img src="/images/devisland/article72/assets/IdentityOnAzure-part8-8.PNG?raw=true" alt="Image not found"/>
</p>

You can create alerts based on specific performance metrics or when certain events are created, the absence of an event or several events are created within a particular time window. For example, alerts can be used to notify you when an average number of sign-in exceeds a certain threshold.

In the [official documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/azure-monitor#create-alerts) you can find a step-by-step guide on how to create a new alert. I encourage you to go through the whole content of this documentation because it shows many useful hints.


# Summary

This was the last article from the Lost in Azure cloud identity series. I hope you enjoyed reading the articles in this series and found them helpful and interesting.
