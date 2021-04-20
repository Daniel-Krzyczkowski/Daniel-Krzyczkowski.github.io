---
title: "Lost in Azure cloud identity - part 1"
excerpt: "This article presents an introduction to the series about using Azure cloud services for identity management"
header:
  image: /images/devisland/article65/assets/IdentityOnAzure-part1-1.png
---

<p align="center">
<img src="/images/devisland/article65/assets/IdentityOnAzure-part1-1.png?raw=true" alt="Lost in Azure cloud identity - part 1"/>
</p>


# Introduction

Once I finished the Cars Island series on my blog (you can check previous articles), I have decided to create series related to identity and access management using Azure cloud services. Important note first - I will focus more on the development side and integration aspects. This series is focused on developers who would like to understand different concepts and mechanisms around identity using Azure cloud services like Azure Active Directory and Azure Active Directory B2C.

It does not matter what kind of application you build, probably always you will touch the identity area. User authentication, authorization, ID Token, Access Token, Scopes, Authority - sounds familiar? In this series we will go through the solution and topics I present below. I hope this series will help you - the developer, understand the integration and concepts behind identity implementation using Azure cloud services.

<p align="center">
<img src="/images/devisland/article65/assets/IdentityOnAzure-part1-5.PNG?raw=true" alt="Image not found"/>
</p>


**Important**

This series assumes that you already have some basic knowledge around identity concepts, application implementation, and Azure cloud services - at least Azure Active Directory.

## What you will find in this series?

We will use below solution architecture to discuss and implement user authentication and authorization. We will see how to use Azure Active Directory and Azure Active Directory B2C to secure applications:

<p align="center">
<img src="/images/devisland/article65/assets/IdentityOnAzure-part1-2.png?raw=true" alt="Image not found"/>
</p>

As you can see above (you can zoom in on the image, or open it in the new tab) there is a sample solution built on the Azure cloud. Let's discuss this architecture in detail because we will focus on each part of it in the next articles.

### Azure Active Directory

Azure Active Directory is used to manage access to corporate applications - in this case to the Tech Mind Factory Corporate Web Application. Employees of Tech Mind Factory company can sign in using their corporate accounts and access web app functionalities. Important fact - these employees can have different application roles assigned. The authorization mechanism implemented in the application prevents access to unauthorized pages once the user role is verified. Roles are injected in the tokens returned from the Azure Active Directory service, once the user is authenticated.

### Azure Active Directory B2C

Tech Mind Factory company has also an application that is available for customers - Tech Mind Factory Customer Application. This is a desktop application (Universal Window Platform app) where customers can register and sign in. To easily manage users and their access to the application, Azure Active Directory B2C service is used. Why second Active Directory? Because we do not want to store user accounts, corporate users and customers, in one directory. What is more, we want to provide flexibility to customers so they can create their accounts themselves and use social media accounts (Facebook) to sign in.

### Tech Mind Factory Shared Web API

Web API is written in ASP .NET Core .NET 5, which returns data for both applications - TMF Customer Application and TMF Corporate Web Application. This Web API is written in a way that enables users to access tokens returned from both identity services - Azure Active Directory and Azure Active Directory B2C. I want to present how to use multiple bearer token authentication schemes.

### Tech Mind Factory Identity Web API

This API, written in ASP .NET Core .NET 5, was created to provide an easy way to create user accounts in the Azure AD B2C using Microsoft Graph API (programmatically) and to provide a migration mechanism. There are some scenarios in the real world where we want to migrate user account from one identity system to another. This API presents how to do it using Azure AD B2C and Microsoft Graph API.

### Tech Mind Factory Identity Monitoring

In the Tech Mind Factory corporation, there is a requirement to monitor Azure AD B2C tenants and collect sign-in and auditing logs. Azure Monitor is used to route Azure Active Directory B2C (Azure AD B2C) sign-in and auditing logs to Log Analytics workspace to analyze data, create dashboards, and alert on specific events.

### Key Vault and Azure Application Insights

Each solution requires good monitoring, this is why the Azure Application Insights service is used to monitor APIs performance and issues. Key Vault is a must-have if we want to store configuration securely in the cloud.


## Source code?

There will be also a dedicated repository on my GitHub shared with the next articles where you will find code samples used in this solution.

<p align="center">
<img src="/images/devisland/article65/assets/IdentityOnAzure-part1-3.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article65/assets/IdentityOnAzure-part1-4.png?raw=true" alt="Image not found"/>
</p>

Below I present the articles which will be included in the series:

### 1. Secure solution with Azure Active Directory and Azure Active Directory B2C

In this article, we are going to focus on two main identity services available in the Azure cloud: Azure AD and Azure AD B2C. First, we will talk about the differences between these two services and when to use each one. We will talk about app registration in these services, scopes, and integration with libraries: Microsoft Identity Web and Microsoft Authentication Library.

### 2. User roles in an application using Azure AD App Roles

In this article, we will discuss authorization concepts and Azure Active Directory App Roles. We will discover how to handle authorization in web applications and how to validate user roles included in the JWT tokens.

### 3. Tailored user experience with Azure AD B2C custom policies and branded pages

In this article, we are going to talk about using Azure Active Directory B2C service to provide a tailored authentication experience for end-users (customers). We will talk about the Identity Experience Framework and custom policies. We will see how to apply custom branding (using HTML and CSS files) for login, and registration pages.

### 4. Custom email verification with SendGrid and Azure AD B2C

In this article, we will talk about integration between email delivery service called SendGrid and Azure Active Directory B2C service. We will discover how to send branded confirmation emails with OTP code when the user provides an email during the registration process.

### 5. User migration to Azure AD B2C using Microsoft Graph API

In this article, we will talk about user migration to Azure Active Directory B2C using Microsoft Graph API. We will talk about migration strategies and see how to create user accounts programmatically.

### 6. DevOps practices for Azure AD B2C custom policies and custom branding

In this article, we are going to focus on the DevOps practices for custom policies and branding files for Azure Active Directory B2C. We will see how to use Azure DevOps to automate the release process.

### 7. Azure Monitor integration with Azure AD B2C for sign-in and audit logs

In the last article, we are going to focus on the audit logs. We will see how to use Azure Monitor to route Azure Active Directory B2C (Azure AD B2C) sign-in and auditing logs to Log Analytics workspace to analyze data, create dashboards, and alert on specific events.


# Summary

In this article, I introduced the Lost in Azure cloud identity series. I hope it will help you start integrating your application solutions with Azure cloud identity services like Azure Active Directory and Azure Active Directory B2C.
