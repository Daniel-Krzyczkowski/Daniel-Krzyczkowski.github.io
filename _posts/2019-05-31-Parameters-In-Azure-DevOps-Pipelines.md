---
title: "PowerShell parameters in the Azure DevOps pipelines"
excerpt: "In this article I would like to present how to use PowerShell Arguments in the Azure DevOps build and release pipelines."
header:
  teaser: "images/devisland/article17/assets/PowerShellParametersAzureDevOps1.PNG?raw=true"
---

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps1.PNG?raw=true" alt="PowerShell parameters in the Azure DevOps pipelines"/>
</p>

Build and release pipelines in the Azure DevOps can have complex structure. Sometimes there is a need to add PowerShell as one of the steps in these pipelines. Why? For instance to update content of the files from the repository or to use some Azure PowerShell cmdlets to make some updates. In this article I would like to present how to use PowerShell Arguments in the Azure DevOps build and release pipelines.

## PowerShell Arguments in the Build pipelines

Let's start from the Build pipelines and PowerShell task. First we have to create sample PowerShell script which will be stored in the GIT repository. We will use this script in the Build piepeline step.
I put this script in the "config" folder located in the "master" branch:

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps3.PNG?raw=true" alt="Image not found"/>
</p>


Below I present the content of this specific script. What we would like to achieve is to retrieve two variables and pass them to the script as parameters.
There are two parameters:

1. ApiManagementServiceName
2. ApiManagementServiceResourceGroup

These are declared in the "Variables" tab in the Build configuration:

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps2.PNG?raw=true" alt="Image not found"/>
</p>


```csharp
[CmdletBinding()]
param (
    $ApiManagementServiceName,
    $ApiManagementServiceResourceGroup
)

$apimServiceName = $ApiManagementServiceName
$resourceGroupName = $ApiManagementServiceResourceGroup

Write-Host "Api Management Service Name: $($apimServiceName)"
Write-Host "Api Management Resource Group Name: $($resourceGroupName)"
```

Now let's analyze build step. For this example lets add only one step with PowerShell script file from the repository above.

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps4.PNG?raw=true" alt="Image not found"/>
</p>

As you can see in the "Arguments" field we have to declare which Variables we want to pass to the PowerShell script as parameters:

```csharp
-ApiManagementServiceName $(ApiManagementServiceName) -ApiManagementServiceResourceGroup $(ApiManagementServiceResourceGroup)
```
Now in the PowerShell script we can get these Variables as params:

```csharp
[CmdletBinding()]
param (
    $ApiManagementServiceName,
    $ApiManagementServiceResourceGroup
)
```

Then we can use them normally in the script:

```csharp
$apimServiceName = $ApiManagementServiceName
$resourceGroupName = $ApiManagementServiceResourceGroup

Write-Host "Api Management Service Name: $($apimServiceName)"
Write-Host "Api Management Resource Group Name: $($resourceGroupName)"
```

Try to run Build and see the result:

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps5.PNG?raw=true" alt="Image not found"/>
</p>

Open step with PowerShell script and see the logs. You should see names declared in the Variables earlier:

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps6.PNG?raw=true" alt="Image not found"/>
</p>

## PowerShell Arguments in the Release pipelines

Steps for the Release pipelines are quite the same. First we have to publish PowerShell script from the repository in the Build artifact.

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps7.PNG?raw=true" alt="Image not found"/>
</p>

Now configure new Release pipeline and connect Build artifact with it:

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps8.PNG?raw=true" alt="Image not found"/>
</p>

Then we have to add Variables that we want to pass:

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps10.PNG?raw=true" alt="Image not found"/>
</p>

Now select PowerShell script from the artifacts:

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps9.PNG?raw=true" alt="Image not found"/>
</p>

Now we have to declare "Arguments" exactly like in the Build pipeline step:

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps11.PNG?raw=true" alt="Image not found"/>
</p>

Creat new release and verify logs:

<p align="center">
<img src="/images/devisland/article17/assets/PowerShellParametersAzureDevOps12.PNG?raw=true" alt="Image not found"/>
</p>


## Summary

In this article I presented how to inject Variables into PowerShell scripts which are the part of the Build and Release pipelines. Of course this is just a simple example but you can also use this method for more advanced scripts and scenarios. For instance you could get secrets from the Azure Key Vault, load them into Variables Groups and then access them from the PowerShell script.