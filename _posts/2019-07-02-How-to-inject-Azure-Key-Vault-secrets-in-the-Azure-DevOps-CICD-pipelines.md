---
title: "How to inject Azure Key Vault secrets in the Azure DevOps CI/CD pipelines"
excerpt: "In this article I would like to present how to inject Azure KeyVault secrets in the Azure DevOps CI/CD pipelines."
---

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault1.png?raw=true" alt="How to inject Azure Key Vault secrets in the Azure DevOps CI/CD pipelines"/>
</p>

Managing secrets in the application is crucial part of the whole development process. Please look at the picture. There are two loops:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault2.PNG?raw=true" alt="Image not found"/>
</p>

* Inner - Focused on the developer teams iterating over their solution development (they consume the configuration published by the outer loop)
* Outer - The Ops Engineer govern the Configuration management and push changes (including Azure KeyVault secrets management)

With such approach you are able keep clear separation of concerns and clean code. What is more, application configuration is much easier to maintain.


## Credentials in the source code

The problem we would like to solve is related with hard-coding connection string to the database in the source code. In this case let me give an example using ASP .NET Core Web API application.
Currently in the "AppSettings.json" file we have database connection string hard-coded:

```csharp
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "AppDatabase": "xxx"
  }
}
```

Our goal is to remove these credentials and inject them in the Build pipeline. You can also ask how developers can then use the app locally?
You will find the answer under <a href="https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-2.2&tabs=windows">this</a> link.


## Azure Key Vault integration with Azure DevOps

First of all we need a Key Vault instance created in Azure:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault3.PNG?raw=true" alt="Image not found"/>
</p>

Once Key Vault is ready we need to add database connection string as a secret:

```csharp
ConnectionStrings:AppDatabase
```

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault4.PNG?raw=true" alt="Image not found"/>
</p>

Please note that our secret has two parts. Because we cannot add ":" character we have to replace it with "--". This will work and during the incjection.
Once the secret is created we can proceed with the next steps.

We have to integrate Azure subscription with our project in the Azure DevOps. In other words - we have to add new service connection. Below I presented steps how to do it:

1. Open "Project settings" tab:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault5.png?raw=true" alt="Image not found"/>
</p>

2. Select "Service connections":

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault6.PNG?raw=true" alt="Image not found"/>
</p>

3. Once connection is ready it should be displayed as shown below:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault7.PNG?raw=true" alt="Image not found"/>
</p>

Now once we have connection estabilished between Azure DevOpS and Azure cloud we can integrate Key Vault.


## Create Variable Group with access to the Key Vault secrets

I have prepared simple build definition for the ASP .NET Core Web API application as shown below:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault8.PNG?raw=true" alt="Image not found"/>
</p>

We will use "Variables" tab in this case to create new variables group. Select "Variable groups" and then "Manage variable groups":

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault9.PNG?raw=true" alt="Image not found"/>
</p>

Next we have to add new variable group so click the button presented below:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault10.PNG?raw=true" alt="Image not found"/>
</p>

In this place we will connect to the Azure Key Vault created before to link secrets in the variable group:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault11.PNG?raw=true" alt="Image not found"/>
</p>

Please note that "Get, List" permissions are required to retrieve secrets so you will have to click "Authorize" button (of course you need entitelents set in the Azure to do it).

Once authorization succeed you should be able to select secrets and link them in the variable groups:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault12.PNG?raw=true" alt="Image not found"/>
</p>

You should see the secret for the database connection string - select it and click "OK" button:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault13.PNG?raw=true" alt="Image not found"/>
</p>

Last step - we have to save all these changes:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault14.PNG?raw=true" alt="Image not found"/>
</p>

Now lets return to the build pipeline definition and select "Variables" tab once again. This time select "Link variable group":

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault15.PNG?raw=true" alt="Image not found"/>
</p>

Select the group we created before and click "Link". Then save changes in the build definition by clicking "Save" button:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault16.PNG?raw=true" alt="Image not found"/>
</p>

Done - secret from the Azure Key Vault are now available for us in the build pipeline. Let do the next step - create PowerShell script to replace connection string in the "AppSettings.json" file with the secrets obtained from the Key Vault.


## Add Replace Tasks task for the credentials replacement and use secrets from the KeyVault

We have to add one more task to the build pipeline definition. This task will be "Replace Tokens" task required to inject secret from the Key Vault in the "AppSettings.json" file. "Replace Tokens" extensions is available fot free to be installed under <a href="https://marketplace.visualstudio.com/items?itemName=qetza.replacetokens">this</a> link.
Once you install it in the Azure DevOps organization please proceed with below steps:

#### Click "+" button in the agent job:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault17.PNG?raw=true" alt="Image not found"/>
</p>

#### Find and select "Replace Tokens" task:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault18.PNG?raw=true" alt="Image not found"/>
</p>

Please note that in the "Advanced" tab there is "token prefix" and "token suffix". We need this information to properly use the tokens in the "AppSettings.json" file. Please also note that "Target files" should be set to "**/*.json":

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault19.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault20.PNG?raw=true" alt="Image not found"/>
</p>

This is the "AppSettings.json" file:

```csharp
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "AppDatabase": "#{ConnectionStrings--AppDatabase}#"
  }
}
```

As you can see I am using token here:

```csharp
#{ConnectionStrings--AppDatabase}#
```
Token name has to be exactly the same as the name of the variable in the Azure DevOps. That's it. Now we can test this one in the Build pipeline.


## Verify the result

Queue the Build pipeline and once it is done, download artifacts and open "AppSettings.json" file. You should see that connection string to the database was injected during the build:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault21.PNG?raw=true" alt="Image not found"/>
</p>

```csharp
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "AppDatabase": "Server=tcp:sampledatabase.database.windows.net,1433;Initial Catalog=app-sql-db;Persist Security Info=False;User ID=test;Password=test1234;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
  }
}

```

Now if you want to use other variables of course you can but you have to declare them in the source code as tokens.

## Summary

In this article I presented how to use Azure Key Vault secrets as a variables in the Azure DevOps to securly manage application secrets and avoid keeping them in the source code. This enables clear separation of concerns and clean code.