---
title: "Azure Hints Series - Azure API Management with custom domain behind Azure Front Door"
excerpt: "This article presents how to setup Azure API Management with custom domain behind Azure Front door"
header:
  image: /images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain.png
---

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain.png?raw=true" alt="Azure Hints Series - Azure API Management with custom domain behind Azure Front Door"/>
</p>

# Introduction

Azure API Management service provides great features to help us protect our set of APIs and access them in a consistent way. When the Azure API Management service is created, it is available under the default domain: *azure-api.net*. However, it is possible to set up a custom domain so API Gateway, Developer Portal, and Management API will be accessible under a custom domain. What is also important to mention is the fact that Azure API Management is a regional service without WAF (Web Application Firewall) capability. This is why in some cases it is valuable to use either Azure Front Door or Azure Application Gateway in front of it.

In this article, I would like to present how to hide the Azure API Management service behind Azure Front Door and how to set up a custom domain that will be used to access Developer Portal, API Gateway, and Management API. In my scenario, I want to use the same custom domains in both services: Azure API Management, and Azure Front Door. You could also implement a scenario where you use specific custom domains for Azure API Management components and other custom domains for Azure Front Door.


## Prerequisites

To follow steps in this article, you will need to have Azure API Management instance. I recommend creating one with *Developer* tier, and when it comes to Azure Front door, I recommend creating one with *Classic* tier. You will also need to create Azure Key Vault to store certificates, and custom domains obtained from one of the third-party domain providers. I have used [GoDaddy](godaddy.com) to register my custom domain, and [ZeroSSL](https://zerossl.com/) to generate certificates. The last thing - I use Azure API Management with *External* VNET integration mode. It means that direct access to Azure API Management is controlled using Network Security Group.

Here is the fragment of the solution architecture:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-20.PNG?raw=true" alt="Image not found"/>
</p>


# Generate certificates for Azure API Management domains


I registered such domain: *tech-mind-factory-identity.org* in the GoDaddy service. I want to achieve below result:

1. Azure API Management Gateway available under *apim.tech-mind-factory-identity.org*
2. Management API of Azure API Management service: *apim.management.tech-mind-factory-identity.org*
3. Developer Portal of Azure API Management: *developer.tech-mind-factory-identity.org*


## Steps to generate free, 90 days certificate with ZeroSSL

Below I present the steps required to generate free, 90 days certificates for our custom domains. For brevity, I present how to generate certificates for a sample domain.

1. In the dashboard select New Certificate:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-02.png?raw=true" alt="Image not found"/>
</p>

2. Type domain name:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-03.png?raw=true" alt="Image not found"/>
</p>

3. Select 90-day certificate option:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-04.png?raw=true" alt="Image not found"/>
</p>

4. Next select Auto-Generate CSR:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-05.png?raw=true" alt="Image not found"/>
</p>


5. Finalize order with free certificate option:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-06.png?raw=true" alt="Image not found"/>
</p>

6. Verify domain ownership by adding CNAME record in your domain registrar (service which owns your custom domain):

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-07.png?raw=true" alt="Image not found"/>
</p>

Once the domain is verified, we can remove above CNAME record. Then we can download the certificate.


## Convert certificate to PFX certificate so it can be uploaded to Azure Key Vault

To be able to upload the certificate to Azure Key Vault, we need to use command to generate *pfx* certificate using private.key file and certificate.crt files:

```text
openssl pkcs12 -export -out apim-cert.pfx -inkey private.key -in certificate.crt

openssl pkcs12 -export -out apim.mgmt-cert.pfx -inkey private.key -in certificate.crt
```

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-08.png?raw=true" alt="Image not found"/>
</p>

Once certificates for APIM Gateway and Management components are generated and converted to PFX, they can be imported to Azure Key Vault. Password is required to be provided for each certificate when adding them to Key Vault. Please note that in my scenario I generated three certificates for specific APIM domains (gateway, developer portal, and management API). This is why I have to upload all three certificates to Azure Key Vault.

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-11.PNG?raw=true" alt="Image not found"/>
</p>




# Configure custom domain with certificate in Azure API Management

Once we upload all certificates to Azure Key Vault, we can setup custom domains and reference certificates. To do it, we have to swtich to *Custom domains* section, and select *Add* button:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-09.PNG?raw=true" alt="Image not found"/>
</p>


In this article, I will present how to set up a custom domain for Azure API Management Gateway but please keep in mind that you setup a custom domain for the developer portal and management API using exactly the same steps but changing the *Type* from the dropdown menu.


<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-10.PNG?raw=true" alt="Image not found"/>
</p>


Here is a configuration for my Azure API Management to use a custom domain for the gateway. As you can see, the certificate is taken from the Azure Key Vault:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-11.PNG?raw=true" alt="Image not found"/>
</p>

It is important to mention that *Default SSL binding* has to be enabled. If you do not set the property, the default certificate is the certificate issued to the default Gateway domain hosted at *azure-api.net.*.

It can take some time to update custom domains so please be patient.

As I mentioned at the beginning, my goal was to achieve below setup:

1. Azure API Management Gateway available under: *apim.tech-mind-factory-identity.org*
2. Management API of Azure API Management service: *apim.management.tech-mind-factory-identity.org*
3. Developer Portal of Azure API Management: *developer.tech-mind-factory-identity.org*


# Hide Azure API Management behind the Azure Front Door

Once we have custom domains configured for Azure API Management Gateway, Management API, and developer portal, we can configure Azure Front Door. Here is the screenshot of my *Front Door Designer*:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-13.PNG?raw=true" alt="Image not found"/>
</p>

Let's discuss this configuration.

## Frontends/domains

First, we have to setup three custom domains to make sure we can route the requests to either the Management API of APIM, Developer Portal, or Gateway. Once we provide the custom domain name by clicking *+* button, we have to verify it by adding CNAME record in the domain registrar's management dashboard:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-14.PNG?raw=true" alt="Image not found"/>
</p>

Here is the example with a custom domain for my API Management Gateway custom domain:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-15.PNG?raw=true" alt="Image not found"/>
</p>

We have to do the same steps for other domains (for Management API, and Developer Portal).

Please note that this is the place where we can apply the Web Application Firewall policy:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-26.PNG?raw=true" alt="Image not found"/>
</p>

In the *Certificate management type* section we can also decide here whether we want to provide our certificate (using Key Vault) or use the managed certificate provided by Azure Front Door.


## Backend pools

For each frontend, we have to configure the backend pool and then configure routing rules. Here is the backend pool for Azure API Management Gateway:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-16.PNG?raw=true" alt="Image not found"/>
</p>

Here is the backend configuration:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-17.PNG?raw=true" alt="Image not found"/>
</p>


## Routing rules

As the last step, we have to configure the routing rule so traffic will go through the Azure Front Door to the specific component of Azure API Management. Here is how it is configured for APIM Gateway routing:

<p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-18.PNG?raw=true" alt="Image not found"/>
</p>


## IMPORTANT!

Probably you are aware but I want to clarify this as it can be confusing. When you use Azure API Management Developer Portal, you can notice in the network trace that it communicates with the Management API of the Azure API Management service. It means that it is not enough to just change the domain for Gateway of APIM, and Developer Portal. To make sure that Developer Portal uses a custom domain for Management API of APIM, you have to properly set *backend host header* in the Azure Front Door for Management API of APIM:

 <p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-19.PNG?raw=true" alt="Image not found"/>
</p>

One last thing, this is the CNAME configuration in my domain registrar. As you can see traffic will be routed to the Azure Front Door:

 <p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-23.PNG?raw=true" alt="Image not found"/>
</p>


# Add restrictions to Network Security Group

As I mentioned at the beginning, an instance of Azure API Management is integrated with Azure Virtual Network using [*External* mode](https://docs.microsoft.com/en-us/azure/api-management/api-management-using-with-vnet?tabs=stv2). With such setup, there is Network Security Group created to control inbound, and outbound traffic for Azure API Management.
To make sure that traffic to APIM Gateway, Developer Portal, and Management API is restricted only for Azure Front Door, in the inbound rules we have to update the rule for ports 80/443.

 <p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-21.PNG?raw=true" alt="Image not found"/>
</p>

Here is how to restrict access using Azure Front Door's service tag:

 <p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-22.PNG?raw=true" alt="Image not found"/>
</p>

You can read more about service tags [here](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview).


# End result

Here is the result. API Management Gateway is available under the custom domain, Management API, and Developer Portal too:

 <p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-24.PNG?raw=true" alt="Image not found"/>
</p>

 <p align="center">
<img src="/images/devisland/article87/assets/azure-hints-02-apim-with-front-door-and-custom-domain-25.PNG?raw=true" alt="Image not found"/>
</p>



# Summary

In this article, I explained how to set up a custom domain for Azure API Management components: Gateway, Management API, and Developer Portal. I also showed how to restrict traffic to Azure API Management from the Azure Front Door only. I hope you found it useful.
