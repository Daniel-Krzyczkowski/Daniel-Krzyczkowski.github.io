---
title: "Manage guest user access with Azure AD External Identities"
excerpt: "This article presents how to use External Identities feature in the Azure AD to manage guest user access"
header:
  image: /images/devisland/article41/assets/AureADExternalIdentities1.png
---

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities1.png?raw=true" alt="Manage guest user access with Azure AD External Identities"/>
</p>

# Introduction

Managing application security and access can be complex. Especially when we would like to provide access not only to the organization's users but also enable partners (or guests) to use our applications. In this article, I would like to present a feature available in the Azure Active Directory called External Identities. With this feature, we can make it possible for guest users to self-sign up and also to sign in with one-time passcode (OTP) without creating a new Microsoft account.

<p align="center">
<img src="/images/devisland/article41/assets/template.png?raw=true" alt="Image not found"/>
</p>

# What is Azure AD External Identities?

External Identities feature allows people outside of our organization to access our apps and resources while letting them sign in using whatever identity they prefer (their private email account). Whether they are part of Azure AD or another IT-managed system or have an unmanaged social identity like Google or Facebook, they can use their own credentials to sign in and use our application.

## Azure AD B2C /= Azure AD External Identities

There is a lot of confusion around differences between Azure AD B2C (business to customer) and Azure AD External Identities. First of all - Azure AD B2C is a stan-alone service that can be created in the Azure cloud. When creating Azure AD B2C, there is a separate Azure AD tenant created underneath. Azure AD B2C provides more customization options. With Azure AD External Identities it is just possible to provide self-sign up for guest users without sending the invitations manually. There is a good comparison of external identities scenarios in the [official documentation](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/compare-with-b2c#external-identities-scenarios).

# External collaboration settings

Recently Microsoft announced few very helpful settings to enable in the Azure AD External Identities. These are:

**Enable Email One-Time Passcode for guests (Preview)**
When enabled, guest users who received the invitation to our organization can sign in using their email and one-time passcode that is sent to this email. In this scenario, Microsoft account is not created (as it was before this option).

**Enable guest self-service sign up via user flows (Preview)**
When enabled, users outside our organization can sign up to obtain access to our applications. This is a very interesting option because we can extend this scenario. We can include custom approvals, so after the guest user signs up, the organization's administrator must accept this request first.

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities2.PNG?raw=true" alt="Image not found"/>
</p>

# One-Time Passcode for guests scenario

Let's talk about the first scenario. We want to invite guest users and enable them to sign in using their email and one-time passcode. In this case, we have to enable this setting in the *External collaboration settings* section in the Azure AD admin portal presented above. Once it is enabled, guest users who received the invitation, after clicking the link, will see the below screens:

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities3.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities4.PNG?raw=true" alt="Image not found"/>
</p>

It is worth to mentioned that pass code is valid for 30 minutes:

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities5.PNG?raw=true" alt="Image not found"/>
</p>


# Enable guest self-service sign up via user flows scenario

Instead of sending invitations to guest users, we can enable self-sign up for them. Once we enable this setting in the Azure AD admin portal presented above on the screenshot, users from the outside of the organization can sign up. Of course, there is an option to add an additional layer with approvals by admin. To enable self-sign up we have to create a user flow. The below steps present how to do it:

**Open User Flows (Preview) tab**

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities6.PNG?raw=true" alt="Image not found"/>
</p>

**Select New user flow button**

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities7.PNG?raw=true" alt="Image not found"/>
</p>

**Setup user flow**

As presented on in the image below, we have to provide the name of the user flow, together with user attributes we want to collect during the registration from the guest user:

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities8.PNG?raw=true" alt="Image not found"/>
</p>

**Assign applications to the user flow**

Once user flow is enabled, we can assign applications to it. Please remember that one application can be associated only with one user flow. Once the application is assigned, when a user visits the application's page, there is a login screen displayed with the option to sign up:

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities10.PNG?raw=true" alt="Image not found"/>
</p>

Guest user can sign up providing the email address:

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities11.PNG?raw=true" alt="Image not found"/>
</p>

Of course during the registration we can collect additional information from the guest users that we checked above during user flow creation:

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities14.PNG?raw=true" alt="Image not found"/>
</p>

Once the guest user accepts the consents, access is granted and the guest user can start using our organization's application.

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities12.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article41/assets/AureADExternalIdentities13.PNG?raw=true" alt="Image not found"/>
</p>


# Summary

In this article, we went through some new features available for the Azure AD External Identities. Now it is much easier to provide access for external users using either self-sign up a scenario or one-time passcode. We can see that Microsoft addresses the need of customers to enable access to their applications for partners. If you want to read more about Azure AD External Identities I encourage you to visit [official website](https://azure.microsoft.com/services/active-directory/external-identities/).
