---
title: "Access Azure Key Vault secrets in the Azure DevOps Release Pipelines"
excerpt: "In this article I would like to present how to inject Azure KeyVault secrets in the Azure DevOps Release pipelines including different environments (Dev, QA and Prod)."
header:
  image: /images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault1.png
---

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault1.png?raw=true" alt="How to inject Azure Key Vault secrets in the Azure DevOps CI/CD pipelines"/>
</p>

Managing secrets in the application is crucial part of the whole development process. Please look at the picture. There are two loops:

<p align="center">
<img src="/images/devisland/article18/assets/AzureDevOpsKeyVault2.PNG?raw=true" alt="Image not found"/>
</p>

* Inner - Focused on the developer teams iterating over their solution development (they consume the configuration published by the outer loop)
* Outer - The Ops Engineer govern the Configuration management and push changes (including Azure KeyVault secrets management)

With such approach you are able keep clear separation of concerns and clean code. What is more, application configuration is much easier to maintain.

In this article I would like to present how integrate Azure Key Vault with Azure DevOps Release pipelines and how to inject secrets per specific environment (in this case Development, QA and Production).


## Prerequisites

Before we start I assume that there are already three separate resource groups created for each enfironment:

* dev-island-app-dev-rg: for Development
* dev-island-app-qa-rg: for QA
* dev-island-app-prod-rg: Production

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault11.PNG?raw=true" alt="Image not found"/>
</p>

Of course there resource groups creation can be also automated. But for this article I created them manually. In each of above resource group I created Key Vault:

* dev-island-app-dev-kv
* dev-island-app-qa-kv
* dev-island-app-prod-kv

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault12.PNG?raw=true" alt="Image not found"/>
</p>

I created separate Key Vault per environment because it is recommended approach by Microsoft:

"Our recommendation is to use a vault per application per environment (Development, Pre-Production and Production)."

You can read more [here.](https://docs.microsoft.com/bs-latn-ba/Azure/key-vault/key-vault-best-practices#use-separate-key-vault)

In each Key Vault I created one secret with different values for each environment:

* Name: "DbConnectionString"

* Value for the Development environment: "Server=(localdb)\\mssqllocaldb;Database=CarsIsland;Trusted_Connection=True;"

* Value for the QA environment: "Server=(localdb)\\mssqllocaldb;Database=CarsIslandQA;Trusted_Connection=True;"

* Value for the Production environment: "Server=(localdb)\\mssqllocaldb;Database=CarsIslandProd;Trusted_Connection=True;"

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault13.PNG?raw=true" alt="Image not found"/>
</p>

**Add new service connection so you can access Azure resources from the Azure DevOps**

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault29.PNG?raw=true" alt="Image not found"/>
</p>


## Prepare release pipeline with Development, QA and Production stages

First of all we have to prepare release pipeline for all three environments: Development, QA and Production. Follo below steps:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault2.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault3.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault4.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault5.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault6.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault7.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault8.PNG?raw=true" alt="Image not found"/>
</p>


## Setup Azure Key Vault integration in the Release pipeline

First of all we have to integrate Key Vault in the Release pipeline so secrets are available through variable group. Each stage in the release pipeline has its own variable group. Lets see how to do it.

**Setup variable group for the Development environment**

Select "Manage variable groups":

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault14.PNG?raw=true" alt="Image not found"/>
</p>

Click "+ Variable group":

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault15.PNG?raw=true" alt="Image not found"/>
</p>

Provide details about this specific variable group:

You have to auhorize Azure DevOps to access Azure subscription and Key Vault:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault16.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault17.PNG?raw=true" alt="Image not found"/>
</p>

Now select which secrets you would like to use as variable in the release pipeline:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault18.PNG?raw=true" alt="Image not found"/>
</p>

In our case we have to select the secret created before called "DbConnectionString":

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault19.PNG?raw=true" alt="Image not found"/>
</p>

Once you select the secret, click "Save" button:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault20.PNG?raw=true" alt="Image not found"/>
</p>

Now get back to the "variable groups" tab and click "Link variable group":

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault10.PNG?raw=true" alt="Image not found"/>
</p>

Select the group we created above so "dev-island-app-dev-kv-vg ". Set "Variable group scope" to "Stages" and select only "Development":

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault21.PNG?raw=true" alt="Image not found"/>
</p>

Click "Save" button:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault22.PNG?raw=true" alt="Image not found"/>
</p>


**Repeat above steps for the QA and Production environments**

Create variable groups for the QA and Production like presented below:

* dev-island-app-qa-kv-vg
* dev-island-app-prod-kv-vg

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault23.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault24.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault26.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault27.PNG?raw=true" alt="Image not found"/>
</p>

Finally it looks like below:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault25.PNG?raw=true" alt="Image not found"/>
</p>

Below three variable groups should be linked:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault28.PNG?raw=true" alt="Image not found"/>
</p>

## Setup Development environment release

We will use ARM template to deploy sample web app with the database connection string added to the configuration. For the Development environment we would like to use database connection string from the "dev-island-app-dev-kv-vg" variable group.

"azuredeploy.json" file looks like below:

```csharp
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "TestAppServicePlanName": {
      "type": "string",
      "minLength": 1
    },
    "TestWebAppName": {
      "type": "string",
      "minLength": 1
    },
    "DbConnectionString": {
      "type": "string",
      "minLength": 1
    },
    "TestAppServicePlanSkuName": {
      "type": "string",
      "defaultValue": "D1",
      "allowedValues": [
        "F1",
        "D1",
        "B1",
        "B2",
        "B3",
        "S1",
        "S2",
        "S3",
        "P1",
        "P2",
        "P3",
        "P4"
      ],
      "metadata": {
        "description": "Describes plan's pricing tier and capacity. Check details at https://azure.microsoft.com/en-us/pricing/details/app-service/"
      }
    }
  },
  "variables": {
    "TestWebAppName": "[concat(parameters('TestWebAppName'), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "name": "[parameters('TestAppServicePlanName')]",
      "type": "Microsoft.Web/serverfarms",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-08-01",
      "sku": {
        "name": "[parameters('TestAppServicePlanSkuName')]"
      },
      "dependsOn": [],
      "tags": {
        "displayName": "TestAppServicePlan"
      },
      "properties": {
        "name": "[parameters('TestAppServicePlanName')]",
        "numberOfWorkers": 1
      }
    },
    {
      "name": "[variables('TestWebAppName')]",
      "type": "Microsoft.Web/sites",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-08-01",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('TestAppServicePlanName'))]"
      ],
      "tags": {
        "[concat('hidden-related:', resourceId('Microsoft.Web/serverfarms', parameters('TestAppServicePlanName')))]": "Resource",
        "displayName": "TestWebApp"
      },
      "properties": {
        "name": "[variables('TestWebAppName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', parameters('TestAppServicePlanName'))]",
        "siteConfig": {
          "connectionStrings": [
            {
              "name": "DbTestConnectionString",
              "connectionString": "[parameters('DbConnectionString')]",
              "type": "string"
            }
          ]
        }
      },
      "resources": [
        {
          "name": "appsettings",
          "type": "config",
          "apiVersion": "2015-08-01",
          "dependsOn": [
            "[resourceId('Microsoft.Web/sites', variables('TestWebAppName'))]"
          ],
          "tags": {
            "displayName": "TestWebAppSettings"
          },
          "properties": {
            "key1": "value1",
            "key2": "value2"
          }
        }
      ]
    }
  ],
  "outputs": {}
}
```

"azuredeploy.parameters.json" file looks like below:

```csharp
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "TestAppServicePlanName": {
      "value": "TestAppServicePlan"
    },
    "TestWebAppName": {
      "value": "TestWebAppCreatedWithARM"
    },
    "DbConnectionString": {
      "value": "#"
    }
  }
}
```

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault30.PNG?raw=true" alt="Image not found"/>
</p>

Please note that in the section called "Override template parameters" we have to replace parameter with the value for the database connection string from the Key Vault:

```csharp
-DbConnectionString $(DbConnectionString)
```


## Setup QA environment release

For the QA environment we will have the same steps like presented above. In this case we are creating web app in the QA resource group in the Azure:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault31.PNG?raw=true" alt="Image not found"/>
</p>


## Setup Production environment release

For the Production environment we will have the same steps like presented above. In this case we are creating web app in the Production resource group in the Azure:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault32.PNG?raw=true" alt="Image not found"/>
</p>


## Create new release and check web apps configuration

Create new release. Once web app is created in each of the resource groups, check "Configuration" tab. You should see that each app has different database connection string retrieved from the Key Vault related with specific environment:

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault33.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault34.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault35.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault36.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article25/assets/AzureDevOpsReleasePipesKeyVault37.PNG?raw=true" alt="Image not found"/>
</p>


## Summary

In this article I explained how to inject Azure Key Vault secrets in the Azure DevOps release pipelines for multiple environments using variable groups. I hope you will find it valuable. Of course instead of variable groups you could use "Add Azure Key Vault" task in the release definition but I wanted to show another approach.