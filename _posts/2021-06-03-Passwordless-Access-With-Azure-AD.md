---
title: "Azure Active Directory passwordless sign-in with FIDO2 Security Keys"
excerpt: "This article presents how to enable passwordless sign-in with FEITIAN BioPass FIDO2 Security Keys"
header:
  image: /images/devisland/article74/assets/AzureAdPasswordless1.png
---

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless1.png?raw=true" alt="Azure Active Directory passwordless sign-in with FIDO2 Security Keys"/>
</p>


# Introduction

In my previous series called *Lost in Azure Cloud Identity* (you can find the first article [here]()) I described how to secure applications with Azure Active Directory and Azure Active Directory B2C identity services. In this article, I would like to present how to access Tech Mind Factory Corporate Application using FIDO2 Security Keys. Thanks to [FEITIAN](https://www.ftsafe.com/) company I was able to receive two BioPass FIDO2 Security Keys for tests. I want to present how to setup these keys with Azure Active Directory to enable passwordless sign-in in the Tech Mind Factory Corporate Application. Source code of the application is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/tree/main/src/tmf-identity-corporate-web-app) and its configuration is described in [this](https://daniel-krzyczkowski.github.io/Lost-In-Azure-Cloud-Identity-Serie-Part2/) previous article.


# FEITIAN BioPass FIDO2 Security Keys

For tests I used [FEITIAN BioPass FIDO2 Security Keys](https://www.ftsafe.com/Products/FIDO/Bio) model K26 (USB-C), and K27 (USB-A). It is worth to mention that FEITIAN is a member of Microsoft Intelligent Security Association (MISA).
<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless1.PNG?raw=true" alt="Image not found"/>
</p>

Configuration is straightforward so let me explain the steps.


# Enable passwordless in the Azure Active Directory

To enable passwordless authentication method with FIDO2, sign in as administrator to your Azure Active Directory tenant. Then follow below steps:

1. From the left menu, select *Security*:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless3.PNG?raw=true" alt="Image not found"/>
</p>

2. Then select *Authentication methods*:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless4.PNG?raw=true" alt="Image not found"/>
</p>

3. In the next blade, select *FIDO2*:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless5.PNG?raw=true" alt="Image not found"/>
</p>

4. Enable the FIDO2 authentication method, decide whether you want to enable it for all users (as in my example). There are also two important points to mention:

* Allow self-service setup - this option should remain set to Yes. If set to no, your users will not be able to register a FIDO key through the MySecurityInfo portal, even if enabled by the Authentication Methods policy.
* Enforce attestation - when this setting is enabled, it requires the FIDO security key metadata to be published and verified with the FIDO Alliance Metadata Service, and also pass Microsoft’s additional set of validation testing

Great, now users can register and manage their FIDO2 Security Keys used to sign in


# User registration and management of FIDO2 security keys

Once the passwordless authentication method is enabled in the Azure Active Directory tenant, we can add our FIDO2 Security Key. Follow the below steps to do it:

### 1. Sign in to [https://myprofile.microsoft.com/](https://myprofile.microsoft.com/)

### 2. Select *Security info*:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless7.PNG?raw=true" alt="Image not found"/>
</p>

### 3. Remember to plug in your FIDO2 Security pass using either USB-A or USB-C device

### 4. Select *Add method*:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless8.PNG?raw=true" alt="Image not found"/>
</p>

### 5. Select *Security key* from the list:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless9.PNG?raw=true" alt="Image not found"/>
</p>

### 6. To setup security key, we have to sign in with two factor authentication (If there is no Azure AD Multi-Factor Authentication method registered, user must add one first). In my case I have Authenticator app installed on my phone:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless10.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless11.PNG?raw=true" alt="Image not found"/>
</p>

### 7. Once request is approved in the Authenticator app, we can select security key type - in this case *USB device* type:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless12.PNG?raw=true" alt="Image not found"/>
</p>

### 8. As I mentioned before, remember to plug in your security key:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless13.PNG?raw=true" alt="Image not found"/>
</p>

### 9. Once we click *Next* button, we are redirected to the page where we can start the process of registering our FIDO2 security key:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless14.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless15.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless16.PNG?raw=true" alt="Image not found"/>
</p>

### 10. First, we have to set the PIN:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless17.PNG?raw=true" alt="Image not found"/>
</p>

Then we are asked to touch the key:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless18.PNG?raw=true" alt="Image not found"/>
</p>

We have to repeat the process. In the last step we have to name our device:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless19.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless20.PNG?raw=true" alt="Image not found"/>
</p>

That's it! Now we can use our FIDO2 device to access Tech Mind Factory Corporate Web App.


# TMF Corporate Web App passwordless sign-in

Here is the Tech Mind Factory Corporate Web App:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless21.PNG?raw=true" alt="Image not found"/>
</p>

After clicking *SIGN IN* button, we are redirected to the Azure AD page. On this page we have to select *Sign-in options*:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless22.PNG?raw=true" alt="Image not found"/>
</p>

Then *Sign in with Windows Hello or security key*:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless23.PNG?raw=true" alt="Image not found"/>
</p>

Then we have to click *Security key*:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless24.PNG?raw=true" alt="Image not found"/>
</p>

Once we provide the PIN and touch the device, we are authenticated:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless25.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless26.PNG?raw=true" alt="Image not found"/>
</p>

We can access app functionalities now:

<p align="center">
<img src="/images/devisland/article74/assets/AzureAdPasswordless27.PNG?raw=true" alt="Image not found"/>
</p>




# Summary

In this article, we discussed how to enable passwordless authentication using FEITIAN Security Keys and Azure Active Directory. If you want to learn more, I encourage you to check [official documentation](https://docs.microsoft.com/en-us/azure/active-directory/authentication/howto-authentication-passwordless-security-key) provided by Microsoft, and [this introduction to passwordless using Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-passwordless) explanation. If you want to check other products from FEITIAN, check [this page](https://www.ftsafe.com/Products).
