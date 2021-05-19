---
title: "Lost in Azure cloud identity - part 7"
excerpt: "This article presents DevOps practices for Azure AD B2C custom policies with branding"
header:
  image: /images/devisland/article71/assets/IdentityOnAzure-part7-1.png
---

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-1.png?raw=true" alt="Lost in Azure cloud identity - part 7"/>
</p>


# Introduction

I have decided to create series related to identity and access management using Azure cloud services. Important note first - I will focus more on the development side and integration aspects. This series is focused on developers who would like to understand different concepts and mechanisms around identity using Azure cloud services like Azure Active Directory and Azure Active Directory B2C. It does not mean that if you are an architect or administrator, you will not find anything interesting. I think that this series can be helpful for everyone who wants to learn more about identity services in the Microsoft Azure cloud.

This is the seventh article from the series. In this article, we are going to talk about DevOps practices for Azure AD B2C custom policies with branded pages. This is an advanced topic and can be (at least for now at the moment of writing this article) applied to custom policies only. We will see how to use Azure DevOps Pipelines to automatically deploy custom policies together with branding assets for customized pages.

**Important**

*This series assumes that you already have some basic knowledge around identity concepts, application implementation, and Azure cloud services - at least Azure Active Directory.*

**Source code**

Source code of all applications and AD B2C custom policies are available on my [GitHub](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity)

**Links to additional helpful resources**

In this specific article, I will focus on the custom policies with branded pages but I will not show how to set up Identity Experience Framework in the Azure Active Directory B2C. There is really good documentation I recommend you to check:

[Tutorial: Create an Azure Active Directory B2C tenant](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant)

[Deploy custom policies with Azure Pipelines](https://docs.microsoft.com/en-us/azure/active-directory-b2c/deploy-custom-policies-devops)

[Azure AD B2C custom policy overview](https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-overview)


**Solution architecture discussed in this series**

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-2.png?raw=true" alt="Image not found"/>
</p>



## Application registration in the Azure AD B2C tenant

First of all, we have to register an application in the Azure AD B2C tenant with specific permission to read and write your organization's trust framework policies in the Azure AD B2C tenant. Here is my registered application:

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-3.PNG?raw=true" alt="Image not found"/>
</p>

We need to add *Policy.ReadWrite.TrustFramework* permission. To do it, select *API permissions*, then *+ Add* permission button, and select *Microsoft Graph*:

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-4.PNG?raw=true" alt="Image not found"/>
</p>

Then select *Application permissions*, and find *Policy.ReadWrite.TrustFramework* permission:

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-5.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-6.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-7.PNG?raw=true" alt="Image not found"/>
</p>


Remember to click *Grant admin consent for Tech Mind Factory* button.


## DevOps flow for the custom policies with branding

The below diagram presents the DevOps flow for custom policies release automation in the Azure DevOps. To simplify the approach I focused on one environment (PROD) in this article, and I published release scripts for this environment on [my GitHub](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/tree/main/src/tmf-identity-ad-b2c/deployment-pipelines). You can extend these release scripts to deploy to all four environments (DEV, TEST, UAT, PROD) as it is presented below on the diagram.

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-8.png?raw=true" alt="Image not found"/>
</p>

Please note that in the diagram there are four tenants (DEV, TEST, UAT, PROD). Let me explain the process of the release pipeline.

1. In the *tmf-identity-ad-b2c* GIT repository there are custom policies stored together with branding assets
2. Once there are changes applied on the *feature* branch, there is a merge done with *develop* branch
3. Once merge is completed, release pipeline is triggered
4. During the release process, PowerShell release scripts are pulled from the Azure DevOps Artifacts Feed (they are stored in the separate *tmf-identity-ad-b2c-release-scripts* GIT repository)
5. Once parameter placeholders are replaced, custom policies are pushed to the DEV Azure AD B2C tenant (from the *develop* branch)
6. Deployments to TEST, UAT, PROD environments are done from the *master* branch. The process is exactly the same but manual approval for the release is required


### PowerShell scripts to automate the release

**Unfortunately, I cannot publish the full source code of PowerShell scripts**

In the separate GIT repository called *tmf-identity-ad-b2c-release-scripts* I keep three PowerShell scripts:

* custom-policies-deployment-script.ps1 - script responsible for deployment of custom policies. You can find initial script in the [official documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/deploy-custom-policies-devops)
* custom-policies-token-transformation-script.ps1 - script responsible for replacing parameter placeholders in the custom policies files (here is the part of the script: *$fileContent = $fileContent -replace "##$($variableName)##", $actualValue*)
* branding-assets-token-transformation-script.ps1 - script responsible for replacing parameter placeholders in the HTML files for branded pages (you can check [*unified.html*](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/main/src/tmf-identity-ad-b2c/branding/idp-pages/template/unified.html) file on my GitHub to see that I use ##StorageAccountPath## placeholder that is replaced with the target Storage account URL during the release process)

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-9.PNG?raw=true" alt="Image not found"/>
</p>

These three scripts are published as feed in the Azure DevOps Artifacts. To publish them to a specific feed in the Azure DevOps Artifacts I configured below *azure-pipeline.yaml* file:

 ```YAML
trigger:
- master

pool:
  vmImage: windows-latest

variables:
- name: ScriptsPath
  value: $(Build.SourcesDirectory)\src\scripts

stages:
- stage: Release
  displayName: Package and Publish
  condition: and(succeeded(), eq(variables['build.sourceBranch'], 'refs/heads/master'))
  jobs: 
    - job: Default
      steps:
        - task: CopyFiles@2
          name: Package
          displayName: Package Flat Files
          inputs:
            sourceFolder: $(ScriptsPath)
            contents: |
              **
            targetFolder: $(Build.ArtifactStagingDirectory)\$(artefactName)-drop
        - task: UniversalPackages@0
          inputs:
            command: 'publish'
            publishDirectory: '$(Build.ArtifactStagingDirectory)\$(artefactName)-drop'
            feedsToUsePublish: 'internal'
            vstsFeedPublish: 'xxx/xxx'
            vstsFeedPackagePublish: '$(artefactName)'
            versionOption: 'patch'
```

With above pipeline I publish PowerShell scripts for custom policies release automation. I keep variables all variables in the variable group in Azure DevOps called **:

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-10.PNG?raw=true" alt="Image not found"/>
</p>


### Custom policies repository with branding files

I keep custom policies together with branding assets in one GIT repository called *tmf-identity-ad-b2c*:

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-11.PNG?raw=true" alt="Image not found"/>
</p>

As you can see above, there are few folders:

* branding - in this folder I keep HTML and CSS files (together with images) for the login, registration, password reset and profile edit pages
* custom-policies - in this folder I keep custom policies files
* deployment-pipelines - in this folder I keep YAML files for Azure DevOps Pipelines (release files)

Each policy file has parameter placeholders added like tenant ID and base policy. Here is an example of [*Signup_Signin.xml*](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/main/src/tmf-identity-ad-b2c/custom-policies/Signup_Signin.xml) policy file:

 ```XML
<TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0" TenantId="##TenantId##"
  PolicyId="##PolicyId##" PublicPolicyUri="http://##TenantId##/##PolicyId##">
  <BasePolicy>
    <TenantId>##TenantId##</TenantId>
    <PolicyId>##BasePolicy##</PolicyId>
  </BasePolicy>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

These parameter placeholders are replaced by the PowerShell script from the Azure DevOps Artifacts Feed. Let's discuss what is happening in the [*azure-pipelines-deployment-template.yml*](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/main/src/tmf-identity-ad-b2c/deployment-pipelines/azure-pipelines-deployment-template.yml):


 ```YAML
jobs:
- deployment: AD_B2C_Custom_Policies_Release
  displayName: AD B2C Custom Policies Release
  pool:
    vmImage: 'VS2017-Win2016'
  environment: ${{parameters.environment}}
  variables:
    scriptsDownloadDirectory: '$(System.DefaultWorkingDirectory)/custom-scripts'
    policyScriptPath: '$(scriptsDownloadDirectory)/custom-policies-deployment-script.ps1'
    customPolicyParameterInjectScriptPath: '$(scriptsDownloadDirectory)/custom-policies-token-transformation-script.ps1'
    brandingAssetsParameterInjectScriptPath: '$(scriptsDownloadDirectory)/branding-assets-token-transformation-script.ps1'
  strategy:
    runOnce:
      deploy:
        steps:
          - checkout: self
            clean: true 
            fetchDepth: 5
            lfs: true
          - task: UniversalPackages@0
            displayName: Getting custom scripts from feed
            inputs:
              command: 'download'
              downloadDirectory: '$(scriptsDownloadDirectory)'
              feedsToUse: 'internal'
              vstsFeed: ${{parameters.vstsFeed}}
              vstsFeedPackage: ${{parameters.vstsFeedPackage}}
              vstsPackageVersion: ${{parameters.vstsFeedPackageVersion}}  
          - task: PowerShell@2
            displayName: Replace parameter placeholders for branding assets
            inputs:
              filePath: '$(brandingAssetsParameterInjectScriptPath)'
              arguments: '-DirectoryPath ${{parameters.brandingFilesDirectory}}'          
          - task: AzureFileCopy@3
            displayName: Copy branding files to Azure Blob Storage
            inputs:
              azureSubscription: ${{parameters.azureSubscription}}
              SourcePath: '${{parameters.brandingFilesDirectory}}'
              Destination: 'AzureBlob'
              storage: '${{parameters.storageAccountName}}'
              ContainerName: '${{parameters.storageAccountContainerName}}'      
          - ${{ each policy in parameters.policies }}:
            - task: PowerShell@2
              displayName: Transform Tokens in ${{policy.path}}
              inputs:
                filePath: '$(customPolicyParameterInjectScriptPath)'
                arguments: '-PolicyId "${{policy.name}}" -File "${{policy.path}}" -DirectoryRoot "$(System.DefaultWorkingDirectory)" -TenantId "${{parameters.tenantId}}" -BasePolicy "${{policy.basePolicy}}"' 
          - ${{ each policy in parameters.policies }}:
            - task: PowerShell@2
              displayName: Release ${{policy.name}}
              inputs:
                filePath: '$(policyScriptPath)'
                arguments: >-
                  -ClientID ${{parameters.clientId}}
                  -ClientSecret ${{parameters.clientSecret}}
                  -TenantId ${{parameters.tenantId}}
                  -PolicyId "${{policy.name}}"
                  -PathToFile "$(System.DefaultWorkingDirectory)/${{policy.path}}"

```

Let's discuss what is happening in the above file:

1. The first task downloads PowerShell scripts from the Azure DevOps Artifacts Feed.
2. The second task replaces storage account name and path to the blob container in the HTML branding files
3. Next task publishes assets to the Azure Blob Storage
4. Then there is an iteration through all custom policies file to replace parameter placeholders with the target values.
5. The last task is responsible for custom policies release to Azure AD B2C using Microsoft Graph API


It is important to mention that I keep all the parameters in the Azure DevOps Variable Group. You can see that I reference two variable groups in the [*azure-pipelines.yml*](https://github.com/Daniel-Krzyczkowski/Lost-In-Azure-Cloud-Identity/blob/main/src/tmf-identity-ad-b2c/deployment-pipelines/azure-pipelines.yml). Here is its structure:

 ```YAML
trigger:
- master

stages:

- stage: DeployCustomPolicies
  displayName: 'Deploy Azure AD B2C custom policies'
  condition: eq(variables['build.sourceBranch'], 'refs/heads/master')
  variables:  
  - group: 'tmf-identity-ad-b2c-release-scripts-vg'
  - group: 'tmf-identity-ad-b2c-custom-policies-release-vg'
  - group: 'tmf-identity-ad-b2c-branding-release-vg'
  jobs:
    - template: azure-pipelines-deployment-template.yml
      parameters:
        environment: 'PROD'
        vstsFeed: $(vstsFeedPublish)
        vstsFeedPackage: $(vstsFeedPackage)
        vstsFeedPackageVersion: $(vstsFeedPackageVersion)
        azureSubscription: $(azureResourceManagerConnectionName)
        storageAccountName: $(storageAccountName)
        storageAccountContainerName: $(storageAccountContainerName)
        brandingFilesDirectory: 'src/branding/idp-pages'
        customPoliciesDirectory: 'src/custom-policies'
        clientId: $(ad-b2c-devops-automation-app-id)
        clientSecret: $(ad-b2c-devops-automation-app-secret)
        tenantId: $(ad-b2c-devops-automation-app-tenant-id)
        sendgrid-email-template-id: $(sendgrid-email-template-id)
        sendgrid-from-email: $(sendgrid-from-email)
        policies:
        - TrustFrameworkBase:
          name: 'B2C_1A_TrustFrameworkBase'
          path: 'src/custom-policies/TrustFrameworkBase.xml' 
        - TrustFrameworkExtensions:
          basePolicy: 'B2C_1A_TrustFrameworkBase'
          name: 'B2C_1A_TrustFrameworkExtensions'
          path: 'src/custom-policies/TrustFrameworkExtensions.xml' 
        - SigninSignIn:
          basePolicy: 'B2C_1A_TrustFrameworkExtensions'
          name: 'B2C_1A_SigninSignUp'
          path: 'src/custom-policies/Signup_Signin.xml'
        - PasswordReset:
          basePolicy: 'B2C_1A_TrustFrameworkExtensions'
          name: 'B2C_1A_PasswordReset'
          path: 'src/custom-policies/PasswordReset.xml'
        - ProfileEdit:
          basePolicy: 'B2C_1A_TrustFrameworkExtensions'
          name: 'B2C_1A_ProfileEdit'
          path: 'src/custom-policies/ProfileEdit.xml'
```

Note that parameters in the above pipeline are injected from the linked variable groups:

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-15.PNG?raw=true" alt="Image not found"/>
</p>



## Final result

Once DevOps Release Pipeline is completed with success, all custom policies and branding assets are published with the right parameters:

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-12.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-13.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article71/assets/IdentityOnAzure-part7-14.PNG?raw=true" alt="Image not found"/>
</p>


# Summary

In this article, we discussed the DevOps approach for the Azure AD B2C custom policies release automation together with branding assets. Automation for custom policies makes your solution more stable and predictable. You can also create additional policies easily basing on the current ones in the repository. With this approach, you also avoid uploading custom policies files and branding assets manually.
