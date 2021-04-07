---
title: "Azure Active Directory Verifiable Credentials - preview introduction"
excerpt: "This article presents introduction to the Azure AD verifiable credentials preview"
header:
  image: /images/devisland/article64/assets/tmf-vc0.jpg
---

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc0.jpg?raw=true" alt="Azure Active Directory Verifiable Credentials - preview introduction"/>
</p>


# Introduction

Azure Active Directory Verifiable Credentials are now in a public preview mode (at the moment of writing this article). You can [visit the official](https://www.microsoft.com/en-us/security/business/identity-access-management/verifiable-credentials) page to read more. On this website, you will find all the details about how to start using Verifiable Credentials with Azure Active Directory.

There is also great [documentation](https://docs.microsoft.com/en-us/azure/active-directory/verifiable-credentials/) with all details required to set up Verifiable Credentials in your own Azure Active Directory tenant.

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc4.PNG?raw=true" alt="Image not found"/>
</p>

# Small proof of concept

Once I discovered that documentation is available, I decided to create a small proof of concept. I have configured Verifiable Credentials accordingly to [details in the documentation.](https://docs.microsoft.com/en-us/azure/active-directory/verifiable-credentials/enable-your-tenant-verifiable-credentials) I have an existing Azure AD B2C tenant so it was much easier because users have to sign in first before they can be issued a verifiable credential.

In this short article, I decided to share the result (I do not want to write another documentation, because the one provided by the Azure AD Team is great) and confirm that this concept works as expected!


## Modified website with QR codes to issue Verifiable Credentials

Below I present modified node.js application which is used to display QR codes:

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc1.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc2.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc3.PNG?raw=true" alt="Image not found"/>
</p>

## Verifiable Credentials in the Microsoft Authenticator App

Below I the user experience in the Microsoft Authenticator App:

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc6.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc7.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc8.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc9.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc10.png?raw=true" alt="Image not found"/>
</p>


## Confirmed DID in the Identity Overlay Network (ION)

Once I created my Verifiable Credential, I verified that it can be found and verified in the ION network. You can read more about it [here](https://techcommunity.microsoft.com/t5/identity-standards-blog/ion-we-have-liftoff/ba-p/1441555).

<p align="center">
<img src="/images/devisland/article64/assets/tmf-vc5.PNG?raw=true" alt="Image not found"/>
</p>


# Summary

In this article, I briefly presented proof of concept related to Verifiable Credentials using Azure Active Directory. In the future, I plan to prepare the blog post series and describe concepts and implementation in detail. If you want to read more about Azure Active Directory Verifiable Credentials, please check [this](https://docs.microsoft.com/en-us/azure/active-directory/verifiable-credentials/decentralized-identifier-overview) documentation.

