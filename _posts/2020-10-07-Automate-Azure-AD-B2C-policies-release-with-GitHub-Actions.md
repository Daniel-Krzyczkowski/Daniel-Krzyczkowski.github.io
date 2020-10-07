---
title: "Automate Azure AD B2C policies release with GitHub Actions"
excerpt: "This article presents how to implement continuous delivery for Azure AD B2C custom policies"
header:
  image: /images/devisland/article43/assets/AdB2CGitHubActionsRelease1.png
---

<p align="center">
<img src="/images/devisland/article43/assets/AdB2CGitHubActionsRelease1.png?raw=true" alt="Automate Azure AD B2C policies release with GitHub Actions"/>
</p>

# Introduction

Azure Active Directory B2C is a great service to secure solution built with the Azure cloud. User authentication is a part of major applications. DevOps practices are crucial for the application release. In this article I would like to present how to use GitHub Actions to configure continuous delivery for the Azure AD B2C custom policies together with custom branding files.

Before we start, here is the final result after release. My customized login and registration experience for users:

<p align="center">
<img src="/images/devisland/article43/assets/AdB2CGitHubActionsRelease1.png?raw=true" alt="Image not found"/>
</p>


# Repository structure

Let's start with the repository structure. There are three main catalogs:

* **branding** - this catalog contains branding files like html, css and images for the AD B2C
* **scripts** - this catalog contains PowerShell scritps responsible for AD B2C cutom policies release together with parameters replacement
* **custom-policies** - AD B2C custom policies files to provide user registration, login, profile edit and password reset capabilities


## custom-policies-branding-deployment-script.ps1

This script is responsible for replacing parameter in the custom policies with the address of the Azure Blob Storage where branding files are located. Please note that we have to pass the parameters at the top of the script:

```powershell
[Cmdletbinding()]
Param(
    [Parameter(Mandatory = $true)][string]$StorageAccountPath,
    [Parameter(Mandatory = $true)][string]$PathToFile
)

Function ReplacePlaceholderWithValueInFile
{
    param( 
        [string]$placeholder,
        [string]$actualValue)

    $customPolicyBrandingFileContent = Get-Content -Path $PathToFile -Raw
    $customPolicyBrandingFileContent -replace $placeholder, $actualValue | Set-Content -Encoding UTF8 -Path $PathToFile
}

ReplacePlaceholderWithValueInFile -placeholder "##STORAGE_ACCOUNT_PATH##" -actualValue $StorageAccountPath
```

## custom-policies-deployment-script.ps1

This script is responsible for getting custom policies content and uploading it to the Azure AD B2C. Please note that we have to pass the parameters at the top of the script:

```powershell
[Cmdletbinding()]
Param(
    [Parameter(Mandatory = $true)][string]$ClientID,
    [Parameter(Mandatory = $true)][string]$ClientSecret,
    [Parameter(Mandatory = $true)][string]$TenantId,
    [Parameter(Mandatory = $true)][string]$PolicyId,
    [Parameter(Mandatory = $true)][string]$PathToFile,
    [Parameter(Mandatory = $true)][string]$ProxyIdentityExperienceFrameworkAppId,
    [Parameter(Mandatory = $true)][string]$IdentityExperienceFrameworkAppId,
    [Parameter(Mandatory = $false)][string]$StorageAccountPath,
    [Parameter(Mandatory = $false)][string]$FacebookClientId
)

Function ReplacePlaceholderWithValueInFile
{
    param( 
        [string]$placeholder,
        [string]$actualValue)

    $customPolicyFileContent = Get-Content -Path $PathToFile -Raw
    $customPolicyFileContent -replace $placeholder, $actualValue | Set-Content -Encoding UTF8 -Path $PathToFile
}

ReplacePlaceholderWithValueInFile -placeholder "##TENANT_ID##" -actualValue $TenantId
ReplacePlaceholderWithValueInFile -placeholder "##ProxyIdentityExperienceFrameworkAppId##" -actualValue $ProxyIdentityExperienceFrameworkAppId
ReplacePlaceholderWithValueInFile -placeholder "##IdentityExperienceFrameworkAppId##" -actualValue $IdentityExperienceFrameworkAppId
ReplacePlaceholderWithValueInFile -placeholder "##STORAGE_ACCOUNT_PATH##" -actualValue $StorageAccountPath
ReplacePlaceholderWithValueInFile -placeholder "##FACEBOOK_CLIENT_ID##" -actualValue $FacebookClientId

try {
    $body = @{grant_type = "client_credentials"; scope = "https://graph.microsoft.com/.default"; client_id = $ClientID; client_secret = $ClientSecret }

    $response = Invoke-RestMethod -Uri https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token -Method Post -Body $body
    $token = $response.access_token

    $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
    $headers.Add("Content-Type", 'application/xml')
    $headers.Add("Authorization", 'Bearer ' + $token)

    $graphuri = 'https://graph.microsoft.com/beta/trustframework/policies/' + $PolicyId + '/$value'
    $policycontent = Get-Content $PathToFile
    $response = Invoke-RestMethod -Uri $graphuri -Method Put -Body $policycontent -Headers $headers

    Write-Host "Policy" $PolicyId "uploaded successfully."
}
catch {
    Write-Host "StatusCode:" $_.Exception.Response.StatusCode.value__

    $_

    $streamReader = [System.IO.StreamReader]::new($_.Exception.Response.GetResponseStream())
    $streamReader.BaseStream.Position = 0
    $streamReader.DiscardBufferedData()
    $errResp = $streamReader.ReadToEnd()
    $streamReader.Close()

    $ErrResp

    exit 1
}

exit 0
```

## Generic custom policies

To enable CI/CD for the custom policies and make them generic, I replaced parameters by the placeholders inside custom policies XML files. Here is the example of "SignUpOrSignin.xml" policy, note that tenant ID is set to "##TENANT_ID##":

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="##TENANT_ID##"
  PolicyId="B2C_1A_SignUpOrSignin"
  PublicPolicyUri="http://##TENANT_ID##/B2C_1A_SignUpOrSignin">

  <BasePolicy>
    <TenantId>##TENANT_ID##</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>

  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="firstName" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="lastName"/>
        <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
        <OutputClaim ClaimTypeReferenceId="country" DefaultValue="" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```


# GitHub Action in action!

We have to setup GitHub Actions to enable possibility for continuous delivery of the Azure AD B2C custom policies together with branding files. Below is the strucure of my GitHub action. What are the steps?

1. Once there is new commit to the main branch, GitHub Action is triggered
2. In the first steps we use PowerShell script described above in the article to update custom policies in the Azure AD B2C
3. Then we have to update branding files to use resources from the Azure Blob Storage (where all html, css and images files are located)
4. At the end we have to push branding files to the Azure Blob Storage


```yml
name: AD-B2C custom policies release
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    env:
      StorageAccountPath: https://stcleanarchb2c.blob.core.windows.net/ad-b2c-branding/branding

    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      - name: Update TrustFrameworkBase.xml
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-deployment-script.ps1 -ClientID ${{secrets.AD_B2C_MANAGEMENT_APP_CLIENT_ID}} -ClientSecret ${{secrets.AD_B2C_MANAGEMENT_APP_CLIENT_SECRET}} -TenantId ${{secrets.AD_B2C_TENANT_ID}} -PolicyId B2C_1A_TrustFrameworkBase -PathToFile .\src\ad-b2c\custom-policies\TrustFrameworkBase.xml -ProxyIdentityExperienceFrameworkAppId ${{secrets.PROXY_IDENTITY_EXPERIENCE_FRAMEWORK_APP_ID}} -IdentityExperienceFrameworkAppId ${{secrets.IDENTITY_EXPERIENCE_FRAMEWORK_APP_ID}} -StorageAccountPath $env:StorageAccountPath
          
      - name: Update TrustFrameworkExtensions.xml
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-deployment-script.ps1 -ClientID ${{secrets.AD_B2C_MANAGEMENT_APP_CLIENT_ID}} -ClientSecret ${{secrets.AD_B2C_MANAGEMENT_APP_CLIENT_SECRET}} -TenantId ${{secrets.AD_B2C_TENANT_ID}} -PolicyId B2C_1A_TrustFrameworkExtensions -PathToFile .\src\ad-b2c\custom-policies\TrustFrameworkExtensions.xml -ProxyIdentityExperienceFrameworkAppId ${{secrets.PROXY_IDENTITY_EXPERIENCE_FRAMEWORK_APP_ID}} -IdentityExperienceFrameworkAppId ${{secrets.IDENTITY_EXPERIENCE_FRAMEWORK_APP_ID}} -FacebookClientId ${{secrets.FACEBOOK_APP_ID}}
      - name: Update SignUpOrSignin.xml
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-deployment-script.ps1 -ClientID ${{secrets.AD_B2C_MANAGEMENT_APP_CLIENT_ID}} -ClientSecret ${{secrets.AD_B2C_MANAGEMENT_APP_CLIENT_SECRET}} -TenantId ${{secrets.AD_B2C_TENANT_ID}} -PolicyId B2C_1A_SignUpOrSignin -PathToFile .\src\ad-b2c\custom-policies\SignUpOrSignin.xml -ProxyIdentityExperienceFrameworkAppId ${{secrets.PROXY_IDENTITY_EXPERIENCE_FRAMEWORK_APP_ID}} -IdentityExperienceFrameworkAppId ${{secrets.IDENTITY_EXPERIENCE_FRAMEWORK_APP_ID}}
          
      - name: Update PasswordReset.xml
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-deployment-script.ps1 -ClientID ${{secrets.AD_B2C_MANAGEMENT_APP_CLIENT_ID}} -ClientSecret ${{secrets.AD_B2C_MANAGEMENT_APP_CLIENT_SECRET}} -TenantId ${{secrets.AD_B2C_TENANT_ID}} -PolicyId B2C_1A_PasswordReset -PathToFile .\src\ad-b2c\custom-policies\PasswordReset.xml -ProxyIdentityExperienceFrameworkAppId ${{secrets.PROXY_IDENTITY_EXPERIENCE_FRAMEWORK_APP_ID}} -IdentityExperienceFrameworkAppId ${{secrets.IDENTITY_EXPERIENCE_FRAMEWORK_APP_ID}}
      - name: Replace Storage Account Path in exception html file
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-branding-deployment-script.ps1 -PathToFile .\src\ad-b2c\branding\template\exception.html -StorageAccountPath $env:StorageAccountPath
  
      - name: Replace Storage Account Path in idpSelector html file
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-branding-deployment-script.ps1 -PathToFile .\src\ad-b2c\branding\template\idpSelector.html -StorageAccountPath $env:StorageAccountPath
      - name: Replace Storage Account Path in multifactor-1.0.0 html file
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-branding-deployment-script.ps1 -PathToFile .\src\ad-b2c\branding\template\multifactor-1.0.0.html -StorageAccountPath $env:StorageAccountPath
          
      - name: Replace Storage Account Path in phoneFactor html file
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-branding-deployment-script.ps1 -PathToFile .\src\ad-b2c\branding\template\phoneFactor.html -StorageAccountPath $env:StorageAccountPath
          
      - name: Replace Storage Account Path in selfAsserted html file
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-branding-deployment-script.ps1 -PathToFile .\src\ad-b2c\branding\template\selfAsserted.html -StorageAccountPath $env:StorageAccountPath
      - name: Replace Storage Account Path in unified html file
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-branding-deployment-script.ps1 -PathToFile .\src\ad-b2c\branding\template\unified.html -StorageAccountPath $env:StorageAccountPath
          
      - name: Replace Storage Account Path in updateProfile html file
        shell: pwsh
        run: |
          .\src\ad-b2c\scripts\custom-policies-branding-deployment-script.ps1 -PathToFile .\src\ad-b2c\branding\template\updateProfile.html -StorageAccountPath $env:StorageAccountPath
      - name: Upload AD B2C branding files
        uses: LanceMcCarthy/Action-AzureBlobUpload@v1.7
        with:
          connection_string: ${{ secrets.AD_B2C_BRANDING_ASSETS_STORAGE_ACCOUNT_CONN_STR }}
          container_name: ad-b2c-branding
          source_folder: src/ad-b2c/branding/
          destination_folder: branding
          clean_destination_folder: true
```

Secrets are kept in the "Secrets" section of GitHub repository:

<p align="center">
<img src="/images/devisland/article43/assets/AdB2CGitHubActionsRelease4.PNG?raw=true" alt="Image not found"/>
</p>


This is the final result of executing above GitHub Action:

<p align="center">
<img src="/images/devisland/article43/assets/AdB2CGitHubActionsRelease3.PNG?raw=true" alt="Image not found"/>
</p>



# Summary

In this article we went through continuous delivery for the Azure AD B2C custom policies using GitHub Actions. It is very helpful to have custom policies generic files because we can easily use them with different Azure AD B2C tenants. If you want to see implementation detalis here is the [link to my GitHub](https://github.com/Daniel-Krzyczkowski/IdentityDeveloperTemplates/blob/master/README.md#Azure-Active-Directory-B2C-generic-templates-with-continuous-delivery) repository.
