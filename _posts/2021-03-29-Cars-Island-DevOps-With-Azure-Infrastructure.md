---
title: "Cars Island - DevOps practices for the Azure infrastructure - part 13"
excerpt: "This article presents how to use some DevOps practices to manage Azure infrastructure"
header:
  image: /images/devisland/article62/assets/CarsIslandInfrastructureDevOps1.jpg
---

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps1.jpg?raw=true" alt="Cars Island - DevOps practices for the Azure infrastructure - part 13"/>
</p>


# Introduction

In [my first article](https://daniel-krzyczkowski.github.io/Cars-Island-Car-Rental-On-Azure-Cloud/), I introduced you to Cars Island car rental on the Azure cloud. I created this fake project to present how to use different Microsoft Azure cloud services and how to their SDKs. I also presented what will be covered in the next articles. Here is the next article from the series where I would like to discuss how to use some DevOps practices to manage Azure infrastructure.

Below I present solution architecture:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps2.png?raw=true" alt="Image not found"/>
</p>


*[Cars Island project is available on my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure)*


# Azure Resource Manager Templates (ARM)

From my [previous article](https://daniel-krzyczkowski.github.io/Cars-Island-Azure-Infrastructure-Copy/) you know that I use Azure Resource Manager Templates (ARM) to create and manage resources created on the Azure cloud. I keep these ARM templates in the separate repository in the Azure DevOps:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps3.PNG?raw=true" alt="Image not found"/>
</p>

Now let's talk first about the changes that are applied to ARM templates and what is then happening in the Azure DevOps.


# Branch policies

Before I continue, a small disclaimer. You can use different branching strategies (like Git flow, GitHub flow, etc.) and still apply the below recommendations to make sure that changes are not directly committed to specific branches and that they are reviewed before the merge is done.

In the Cars Island GIT repository for ARM templates I have one branch: *master*. This branch has policies enabled so direct commits are not allowed:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps4.png?raw=true" alt="Image not found"/>
</p>

For the *master* branch I applied the below policies:

### Require a minimum number of reviewers

First of all - when there are any changes, they cannot be committed directly. A new pull request should be created to discuss what was changed. At least one person from the team has to review the changes and either approve them or reject them. Once changes are approved but someone will add anything new to the source branch (*feature* branch), approval will be reset and the reviewer has to review the changes again:


### Check for linked work items

As you remember from my [previous article](https://daniel-krzyczkowski.github.io/Cars-Island-Backlog-Structure-And-Definition/), I created a backlog in Azure DevOps to control changes. In this backlog, there was dedicated *Epic* for the infrastructure. Each time there is a new pull request created, work items from this backlog have to be attached. Why? Because it is much easier to discover what kind of changes are applied.

### Check for comment resolution

During the code review process, the reviewer can comment on some changes. The creator of the pull request has resolved these comments before the merge is possible. This prevents omitting the comments with important notes and requirements.


### Limit merge types

We can control how commits history is preserved in the GIT repository. In this case, I applied *Basic merge (no fast-forward)* type because I want to keep each commit in the history tree.


<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps5.PNG?raw=true" alt="Image not found"/>
</p>


Now let's talk about the release pipeline and variables.


# Azure DevOps Pipeline Variable Groups

Azure DevOps Pipeline Variable Groups can be used to store variables used in the different pipelines. As you can see I have three variable groups created - each one keeps variables for different environment - DEV, TEST, and PROD:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps6.PNG?raw=true" alt="Image not found"/>
</p>

Let's talk about variable group created for DEV environment. As you can see I declared many different variables:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps7.PNG?raw=true" alt="Image not found"/>
</p>

We can mark some variable values to be secrets - then they will not appear in the pipeline's logs and they will be hidden:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps8.png?raw=true" alt="Image not found"/>
</p>

**Important**

As you can see in the picture above, there is an option to link the secret from the Azure Key Vault. This option can be helpful when you have a separate, initial Key Vault, where you store values for the infrastructure that will be created. Example? You can store the username and password for the Azure SQL database there. Then this information can be injected into the ARM templates during the execution of the release pipeline. In Cars Island the process is simplified. I store secrets directly in the Azure DevOps variable groups.


# Azure DevOps Environments

An environment in the Azure DevOps is a collection of resources, that can be targeted by deployments from a pipeline. Typical examples of environment names are Dev, Test, QA, Staging, and Production. I created three environments that I will reference in the release pipeline:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps9.PNG?raw=true" alt="Image not found"/>
</p>

I encourage you to read [this documentation](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops) to learn more about declaring environments.


# Azure DevOps Pipeline Templates

Before I continue I would like to recommend watching my course called [Microsoft DevOps Solutions: Implementing Orchestration Automation Solutions](https://app.pluralsight.com/library/courses/microsoft-devops-solutions-implementing-orchestration-automation-solutions/table-of-contents) where I described in details some aspects mentioned in this article.

In the GIT repository for ARM templates, I created a separate folder called *pipelines* where I store YAML files with a declaration for the infrastructure release. In the Azure DevOps you can of course use visual designer but recommended approach is to use YAML files to declare builds and releases as a code:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps10.PNG?raw=true" alt="Image not found"/>
</p>

## arm-deployment-template.yml

```json
jobs:

  - deployment: DeployRG
    environment: ${{parameters.environment}}
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
            clean: true 
            fetchDepth: 5
            lfs: true
          - task: AzureResourceManagerTemplateDeployment@3
            displayName: Deploy Resource Group
            inputs:
              deploymentScope: 'Subscription'
              azureResourceManagerConnection: ${{parameters.azureResourceManagerConnectionName}}
              subscriptionId: ${{parameters.targetSubscriptionId}}
              location: 'West Europe'
              templateLocation: 'Linked artifact'
              csmFile: '$(System.DefaultWorkingDirectory)/arm/rg-azure-deploy.json'
              overrideParameters: '-rgName ${{parameters.resourceGroupName}} -location ${{parameters.resourceGroupLocation}}'
              deploymentMode: 'Incremental'
  
  - deployment: DeployResourcesIntoRG
    condition: succeeded()
    dependsOn: DeployRG
    environment: ${{parameters.environment}}
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
            clean: true 
            fetchDepth: 5
            lfs: true         
          - task: AzureResourceManagerTemplateDeployment@3
            displayName: Deploy resources into resource group
            inputs:
              deploymentScope: 'Resource Group'
              azureResourceManagerConnection: ${{parameters.azureResourceManagerConnectionName}}
              subscriptionId: ${{parameters.targetSubscriptionId}}
              action: 'Create Or Update Resource Group'
              resourceGroupName: ${{parameters.resourceGroupName}}
              location: 'West Europe'
              templateLocation: 'Linked artifact'
              csmFile: '$(System.DefaultWorkingDirectory)/arm/azure-deploy.json'
              csmParametersFile: '$(System.DefaultWorkingDirectory)/arm/azure-deploy.parameters.${{parameters.env}}.json'
              overrideParameters: '-sendgrid_name "${{parameters.sendGridName}}" -sendgrid_password "${{parameters.sendGridPublisherPassword}}" -sendgrid_email "${{parameters.sendGridPublisherEmail}}" -sendgrid_firstName "${{parameters.sendGridPublisherFirstName}}" -sendgrid_lastName "${{parameters.sendGridPublisherLastName}}" -sendgrid_company "${{parameters.sendGridCompanyName}}" -sendgrid_website "${{parameters.sendGridWebsite}}"'
              deploymentMode: 'Incremental'

  - deployment: ApiManagementDeployment
    environment: ${{parameters.environment}}
    dependsOn: DeployResourcesIntoRG
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
            clean: true 
            fetchDepth: 5
            lfs: true
          - task: AzureResourceManagerTemplateDeployment@3
            displayName: Deploy Api Management resource
            inputs:
              deploymentScope: 'Resource Group'
              azureResourceManagerConnection:  ${{parameters.azureResourceManagerConnectionName}}
              subscriptionId: ${{parameters.targetSubscriptionId}}
              action: 'Create Or Update Resource Group'
              resourceGroupName: ${{parameters.resourceGroupName}}
              location: ${{parameters.resourceGroupLocation}}
              templateLocation: 'Linked artifact'
              csmFile: '$(System.DefaultWorkingDirectory)/arm/apim-azure-deploy.json'
              overrideParameters: '-api_management_name "${{parameters.apiManagementName}}" -api_management_publisher_email "${{parameters.apiManagementPublisherEmail}}" -api_management_publisher_name "${{parameters.apiManagementPublisherName}}" -sku {"name": "${{parameters.apiManagementSku}}", "capacity": 1} -resourceTags {"Environment": "${{parameters.apiManagementEnvironmentTag}}", "Project": "${{parameters.apiManagementProjectTag}}"}'
              deploymentMode: 'Incremental'
```

The template above contains three deployments:

### DeployRG - to create new resource group (if one does not already exist):

```json
  - deployment: DeployRG
    environment: ${{parameters.environment}}
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
            clean: true 
            fetchDepth: 5
            lfs: true
          - task: AzureResourceManagerTemplateDeployment@3
            displayName: Deploy Resource Group
            inputs:
              deploymentScope: 'Subscription'
              azureResourceManagerConnection: ${{parameters.azureResourceManagerConnectionName}}
              subscriptionId: ${{parameters.targetSubscriptionId}}
              location: 'West Europe'
              templateLocation: 'Linked artifact'
              csmFile: '$(System.DefaultWorkingDirectory)/arm/rg-azure-deploy.json'
              overrideParameters: '-rgName ${{parameters.resourceGroupName}} -location ${{parameters.resourceGroupLocation}}'
              deploymentMode: 'Incremental'
```

Please note that *deploymentMode* is set to *Incremental*. With this configuration, we can avoid re-creating resource groups (or other Azure resources if they already exist - names are verified first).

There is also *overrideParameters* section where we can pass the parameters to override the one used in the parameter files (like [azure-deploy.parameters.dev.json](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure/blob/master/src/arm-templates/azure-deploy.parameters.dev.json)). Above parameters are taken from the variable group (connection to variable groups will be mentioned below).

For the *csmFile* section we have to provide the path to the ARM json file (for instance for the resource group this one: *rg-azure-deploy.json*).


### DeployResourcesIntoRG - to create resources in the resouce group (if these resources do not already exist):

```json
  - deployment: DeployResourcesIntoRG
    condition: succeeded()
    dependsOn: DeployRG
    environment: ${{parameters.environment}}
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
            clean: true 
            fetchDepth: 5
            lfs: true         
          - task: AzureResourceManagerTemplateDeployment@3
            displayName: Deploy resources into resource group
            inputs:
              deploymentScope: 'Resource Group'
              azureResourceManagerConnection: ${{parameters.azureResourceManagerConnectionName}}
              subscriptionId: ${{parameters.targetSubscriptionId}}
              action: 'Create Or Update Resource Group'
              resourceGroupName: ${{parameters.resourceGroupName}}
              location: 'West Europe'
              templateLocation: 'Linked artifact'
              csmFile: '$(System.DefaultWorkingDirectory)/arm/azure-deploy.json'
              csmParametersFile: '$(System.DefaultWorkingDirectory)/arm/azure-deploy.parameters.${{parameters.env}}.json'
              overrideParameters: '-sendgrid_name "${{parameters.sendGridName}}" -sendgrid_password "${{parameters.sendGridPublisherPassword}}" -sendgrid_email "${{parameters.sendGridPublisherEmail}}" -sendgrid_firstName "${{parameters.sendGridPublisherFirstName}}" -sendgrid_lastName "${{parameters.sendGridPublisherLastName}}" -sendgrid_company "${{parameters.sendGridCompanyName}}" -sendgrid_website "${{parameters.sendGridWebsite}}"'
              deploymentMode: 'Incremental'
```

Once the resource group is created, the next deployment can be run - *DeployResourcesIntoRG*. Please note that there is a *dependsOn: DeployRG* section. With the [conditions in Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/conditions?view=azure-devops&tabs=yaml) we can decide whether next deployment should be executed or not basing on the previous one's status (if it is successfull, next deployment can be executed).


### ApiManagementDeployment - to create Azure API Management service (if it does not already exist)

```json
  - deployment: ApiManagementDeployment
    environment: ${{parameters.environment}}
    dependsOn: DeployResourcesIntoRG
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
            clean: true 
            fetchDepth: 5
            lfs: true
          - task: AzureResourceManagerTemplateDeployment@3
            displayName: Deploy Api Management resource
            inputs:
              deploymentScope: 'Resource Group'
              azureResourceManagerConnection:  ${{parameters.azureResourceManagerConnectionName}}
              subscriptionId: ${{parameters.targetSubscriptionId}}
              action: 'Create Or Update Resource Group'
              resourceGroupName: ${{parameters.resourceGroupName}}
              location: ${{parameters.resourceGroupLocation}}
              templateLocation: 'Linked artifact'
              csmFile: '$(System.DefaultWorkingDirectory)/arm/apim-azure-deploy.json'
              overrideParameters: '-api_management_name "${{parameters.apiManagementName}}" -api_management_publisher_email "${{parameters.apiManagementPublisherEmail}}" -api_management_publisher_name "${{parameters.apiManagementPublisherName}}" -sku {"name": "${{parameters.apiManagementSku}}", "capacity": 1} -resourceTags {"Environment": "${{parameters.apiManagementEnvironmentTag}}", "Project": "${{parameters.apiManagementProjectTag}}"}'
              deploymentMode: 'Incremental'
```

The last deployment is created separately for the API Management deployment. Once we have the deployment YAML file defined, we can move to the *azure-pipelines.yml* file structure below.

## arm-deployment-template.yml

This file contains stages which uses the above deployment template to create resources for DEV, TEST, and PROD environment:

```json
trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: Deploy_DEV_environment_infrastructure
  displayName: 'Create Cars Island Azure resources for DEV environment'
  variables:
  - group: 'cars-island-infrastructure-dev-env-variable-group'
  jobs:
  - template: arm-deployment-template.yml
    parameters:
      azureResourceManagerConnectionName: '$(azureResourceManagerConnectionName)'
      targetSubscriptionId: '$(targetSubscriptionId)'
      resourceGroupName: '$(resourceGroupName)'
      resourceGroupLocation: '$(resourceGroupLocation)'
      apiManagementName: '$(apiManagementName)'
      apiManagementPublisherEmail: '$(apiManagementPublisherEmail)'
      apiManagementPublisherName: '$(apiManagementPublisherName)'
      apiManagementSku: '$(apiManagementSku)'
      apiManagementEnvironmentTag: '$(apiManagementEnvironmentTag)'
      apiManagementProjectTag: '$(apiManagementProjectTag)'
      sendGridName: '$(sendGridName)'
      sendGridPublisherFirstName: '$(sendGridPublisherFirstName)'
      sendGridPublisherLastName: '$(sendGridPublisherLastName)'
      sendGridPublisherEmail: '$(sendGridPublisherEmail)'
      sendGridPublisherPassword: '$(sendGridPublisherPassword)'
      sendGridCompanyName: '$(sendGridCompanyName)'
      sendGridWebsite: '$(sendGridWebsite)'
      env: '$(env)'
      environment: 'DEV'

- stage: Deploy_TEST_environment_infrastructure
  displayName: 'Create Cars Island Azure resources for TEST environment'
  variables:
  - group: 'cars-island-infrastructure-test-env-variable-group'
  jobs:
  - template: arm-deployment-template.yml
    parameters:
      azureResourceManagerConnectionName: '$(azureResourceManagerConnectionName)'
      targetSubscriptionId: '$(targetSubscriptionId)'
      resourceGroupName: '$(resourceGroupName)'
      apiManagementName: '$(apiManagementName)'
      apiManagementPublisherEmail: '$(apiManagementPublisherEmail)'
      apiManagementPublisherName: '$(apiManagementPublisherName)'
      apiManagementSku: '$(apiManagementSku)'
      apiManagementEnvironmentTag: '$(apiManagementEnvironmentTag)'
      apiManagementProjectTag: '$(apiManagementProjectTag)'
      sendGridName: '$(sendGridName)'
      sendGridPublisherFirstName: '$(sendGridPublisherFirstName)'
      sendGridPublisherLastName: '$(sendGridPublisherLastName)'
      sendGridPublisherEmail: '$(sendGridPublisherEmail)'
      sendGridPublisherPassword: '$(sendGridPublisherPassword)'
      sendGridCompanyName: '$(sendGridCompanyName)'
      sendGridWebsite: '$(sendGridWebsite)'
      env: '$(env)'
      environment: 'TEST'

- stage: Deploy_PROD_environment_infrastructure
  displayName: 'Create Cars Island Azure resources for PROD environment'
  variables:
  - group: 'cars-island-infrastructure-prod-env-variable-group'
  jobs:
  - template: arm-deployment-template.yml
    parameters:
      azureResourceManagerConnectionName: '$(azureResourceManagerConnectionName)'
      targetSubscriptionId: '$(targetSubscriptionId)'
      resourceGroupName: '$(resourceGroupName)'
      apiManagementName: '$(apiManagementName)'
      apiManagementPublisherEmail: '$(apiManagementPublisherEmail)'
      apiManagementPublisherName: '$(apiManagementPublisherName)'
      apiManagementSku: '$(apiManagementSku)'
      apiManagementEnvironmentTag: '$(apiManagementEnvironmentTag)'
      apiManagementProjectTag: '$(apiManagementProjectTag)'
      sendGridName: '$(sendGridName)'
      sendGridPublisherFirstName: '$(sendGridPublisherFirstName)'
      sendGridPublisherLastName: '$(sendGridPublisherLastName)'
      sendGridPublisherEmail: '$(sendGridPublisherEmail)'
      sendGridPublisherPassword: '$(sendGridPublisherPassword)'
      sendGridCompanyName: '$(sendGridCompanyName)'
      sendGridWebsite: '$(sendGridWebsite)'
      env: '$(env)'
      environment: 'PROD'
```

As you can see, each stage uses a different variable group (like DEV stage uses *cars-island-infrastructure-dev-env-variable-group* variable group). Once these variable groups are linked to the pipeline, parameters can be injected and used in the YAML templates. How can we link these variable groups then?


# Linking Variable Groups with Pipelines

Once we define new pipeline in the Azure DevOps, we can link variable groups so we can inject values for parameters presented above in the templates. To do it, click *Edit* button:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps11.PNG?raw=true" alt="Image not found"/>
</p>

Then select *Triggers*:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps12.png?raw=true" alt="Image not found"/>
</p>

Then switch to the *Variables* section and select *Variable groups*. From this place you can link any variable group to use variables in the pipeline:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps13.PNG?raw=true" alt="Image not found"/>
</p>


# Run release for the infrastructure

Once we have steps completed above, we can run the release pipeline. Please note that in this case all three environments will be created: DEV, TEST, and PROD:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps14.PNG?raw=true" alt="Image not found"/>
</p>

What about scenarios in which we want to control when there is a deployment done for a specific environment (like for the production once)? In this case, we have to use gates.


# Approvals and checks

As you remember, I defined three environments in the Azure DevOps mentioned above in the article: DEV, TEST, and PROD. Now for each environment, I can define approvals and checks. To do it, we have to open specific environment, like PROD, and select three dots, and then select *Approvals and checks*:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps15.png?raw=true" alt="Image not found"/>
</p>


There is a whole list of available checks to select from:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps16.PNG?raw=true" alt="Image not found"/>
</p>

You can select *Approvals* and then decide who is responsible for the final approval before there is deployment executed:

<p align="center">
<img src="/images/devisland/article62/assets/CarsIslandInfrastructureDevOps17.PNG?raw=true" alt="Image not found"/>
</p>

Each time there is a new release available, the person indicated above will have to manually approve the execution of the deployment stage.


# Summary

In this article, I described how to use Azure DevOps to implement the DevOps process for the Azure infrastructure release. As you can see, there are many great features you can use to make your process more stable. Once again, I encourage you to watch my Pluralsight course called [Microsoft DevOps Solutions: Implementing Orchestration Automation Solutions](https://app.pluralsight.com/library/courses/microsoft-devops-solutions-implementing-orchestration-automation-solutions/table-of-contents) to learn more.
