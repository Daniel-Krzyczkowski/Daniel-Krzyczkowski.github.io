---
title: "Azure AD B2C Series Custom Policies with custom claims"
excerpt: "In this article would like to present how to use custom claims in the Azure AD B2C Custom Policies (Identity Experience Framework)."
---

<p align="center">
<img src="/images/devisland/article22/assets/B2cSeriesCustomClaims1.png?raw=true" alt="Azure AD B2C Series Custom Policies with custom claims"/>
</p>

I had a chance to work with the Azure Active Directory B2C quite a lot recently and decided that it would be nice to share some knowledge about it. Just to make life easier for people using it especially when there are some custom usage scenarios. This is the first article from the series and in this article I would like to present how to use custom claims (custom attributes) in the Azure AD B2C Identity Experience Framework Custom Policies.

There are also links to the great content after opening "Identity Experience Framework" tab in the Azure portal:

<p align="center">
<img src="/images/devisland/article22/assets/B2cSeriesCustomClaims2.gif?raw=true" alt="Image not found"/>
</p>


## Introduction and assumptions

Before we start I assume that you are familiar with the Azure AD B2C Identity Experience Framework and initial setup is done. If you would like to start I recommend to check [official documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-get-started-custom). In this series we will use already configured AD B2C tenant with Custom Policies. After all steps from the documentation are completed we are ready to move forward.

**We will use "SocialAndLocalAccounts" policies files from the starter pack.**

Below files will be used:

* TrustFrameworkBase.xml
* TrustFrameworkExtensions.xml
* SignUpOrSignin.xml
* ProfileEdit.xml
* PasswordReset.xml


## Custom claims (attributes)

There is a list of built-in claims (attributes) for user entity in the Azure AD B2C:

* objectId
* accountEnabled
* city
* country
* displayName
* givenName
* jobTitle
* otherMails
* postalCode
* signInNames
* signInNames.emailAddress
* signInNames.userName
* state
* streetAddress
* surname
* userIdentities

They can be found in the [official documentation](https://docs.microsoft.com/en-us/previous-versions/azure/ad/graph/api/entity-and-complex-type-reference#user-entity).

**What about custom attributes?**