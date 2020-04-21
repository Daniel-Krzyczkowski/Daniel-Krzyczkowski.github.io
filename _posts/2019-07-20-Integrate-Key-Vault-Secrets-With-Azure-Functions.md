---
title: "Integrate Key Vault Secrets With Azure Functions"
excerpt: "In this article I would like to present how to integrate Azure Functions with Key Vault to inject secrets in the settings."
header:
  image: /images/devisland/article19/assets/FunctionAppAndKeyVault1.png
---

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault1.png?raw=true" alt="Build IoT Solutions with Azure and Serverless"/>
</p>

There are a lot of different scenarios where Azure Functions can be used. Interation with other Azure components is possible but we can also call external services. The question in how to secure credentials and store them in the right place - in this case Azure Key Vault. In this article I would like to present how to integrate Azure Key Vault with Azure Functions to access Key Vault secrets from the source code in the secure way.


## Create Azure Key Vault and Azure Function App

First of all we have to create sample Key Vault and Azure Function App. Below here are my two resources created:

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault2.PNG?raw=true" alt="Image not found"/>
</p>


## Add secrets to the Azure Key Vault

Credentials should be stored in the secure way using Azure Key Vault secrets. Lets add two secrets:
* Username: sampleazure@com
* Password: Test1234@

We will use these two secrets in the Azure Function later.

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault3.PNG?raw=true" alt="Image not found"/>
</p>

Below is the simple Http Trigger Function App I created. We will modify its code below to inject secrets.

```csharp
public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

## Enable system-asigned managed identity for the Function App

Before we can use Azure Key Vault secrets in the Azure Function code, we have to assign a Managed Identity to it. If you are not familiar with Managed Identities, I encourage you to read more in [this](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) article.

Navigate to the "Platform features" tab and select "Identity":

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault4.PNG?raw=true" alt="Image not found"/>
</p>

Set "Status" to "On" and then click "Save" button - confirm information displayed in the dialog:

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault5.PNG?raw=true" alt="Image not found"/>
</p>

There should be confirmation displayed that resource was registered in the Azure Active Directory:

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault6.PNG?raw=true" alt="Image not found"/>
</p>


## Add Key Vault access policy for the Function App

Now we need to add access policy for the Function App so it will be able to read secrets from the Azure Key Vault.

Select "Access policies" tab:

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault7.PNG?raw=true" alt="Image not found"/>
</p>

Find Function and select it in the "Service Principal" section. Please note that we need to select "Get" and "List" permissions:

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault8.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault9.PNG?raw=true" alt="Image not found"/>
</p>

Click "Save" button:

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault10.PNG?raw=true" alt="Image not found"/>
</p>

## Add Key Vault secrets reference in the Function App configuration

Now we need to refer to the Key Vault secrets in the Function App configuration.
Navigate to the "Secrets" tab in the Key Vault and copy "Secret Identifier" for both "Username" and "Password" secrets. We will use them soon:

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault11.PNG?raw=true" alt="Image not found"/>
</p>

Now we wil add these identifiers to the "Configuration" of the Function App.
There will be two settings added:

UsernameFromKeyVault:

@Microsoft.KeyVault(SecretUri={copied identifier for the username secret})

PasswordFromKeyVault:

@Microsoft.KeyVault(SecretUri={copied identifier for the password secret})

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault12.PNG?raw=true" alt="Image not found"/>
</p>


## Test secrets injection

Now we need to modify the source code of the Function App. The quickest way to check if secrets injection works is to use logger.
We will try to display injected secrets.

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;
using System;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    var username =  Environment.GetEnvironmentVariable("UsernameFromKeyVault", EnvironmentVariableTarget.Process);
    var password =  Environment.GetEnvironmentVariable("PasswordFromKeyVault", EnvironmentVariableTarget.Process);

    log.LogInformation($"Username: {username}");
    log.LogInformation($"Username: {password}");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

Save and run the function. Look at the logs - secrets should be displayed there:

<p align="center">
<img src="/images/devisland/article19/assets/FunctionAppAndKeyVault13.PNG?raw=true" alt="Image not found"/>
</p>


## Summary

In this article I presented how to securly pass secrets from the Azure Key Vault to the Azure Function App. With this solution we are not storing credentials directly in the configuration but we inject it from the Key Vault.
