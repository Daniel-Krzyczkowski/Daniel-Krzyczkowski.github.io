---
title: "Azure AD B2C Series - Custom Policies release automation with Azure DevOps"
excerpt: "In this article would like to present how to setup release pipeline in Azure DevOps to automatically update custom policies in the Azure AD B2C (Identity Experience Framework)."
---

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease1.png?raw=true" alt="Azure AD B2C Series Custom Policies release automation with Azure DevOps"/>
</p>

I had a chance to work with the Azure Active Directory B2C quite a lot recently and decided that it would be nice to share some knowledge about it. Just to make life easier for people using it especially when there are some custom usage scenarios. This is the fourth article from the series and in this article I would like to present how to setup release pipeline in Azure DevOps to automatically update custom policies in the Azure AD B2C Identity Experience Framework.

There are also links to the great content after opening "Identity Experience Framework" tab in the Azure portal:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease2.PNG?raw=true" alt="Image not found"/>
</p>


## Introduction and assumptions

Before we start I assume that you are familiar with the Azure AD B2C Identity Experience Framework and initial setup is done. If you would like to start I recommend to check [official documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-get-started-custom). In this series we will use already configured AD B2C tenant with Custom Policies. After all steps from the documentation are completed we are ready to move forward.

## Prepare Azure AD B2C for automatic updates of custom policies ##

We are going to access and update custom policies using Microsoft Graph API. To be able to do it we need to register new application in the Azure Active Directory (in Azure AD B2C tenant) and assign the right permissions. Let's see how to do it.

### Register new application in the Azure Active Directory ###

In the tenant of your Azure AD B2C in the Azure portal select "Azure Active Directory" tab:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease3.PNG?raw=true" alt="Image not found"/>
</p>

Check Tenant ID - we will need it to setup release pipeline in the Azure DevOps:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease3.PNG?raw=true" alt="Image not found"/>
</p>

Select "App registrations (Legacy) and click "New application registration":

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease4.PNG?raw=true" alt="Image not found"/>
</p>

Provide the name for the app and set fields as presented below:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease5.PNG?raw=true" alt="Image not found"/>
</p>

Once app is created, copy "Application ID" - we will need it to setup release pipeline in the Azure DevOps:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease6.PNG?raw=true" alt="Image not found"/>
</p>

Then open "Required permissions" tab and click "Add" button:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease7.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease8.PNG?raw=true" alt="Image not found"/>
</p>

Select "Microsoft Graph" and select checkbox under "Application Permissions" called "Read and write your organization's trust framework policies":

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease9.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease10.PNG?raw=true" alt="Image not found"/>
</p>

Then click "Grant permissions" button:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease11.PNG?raw=true" alt="Image not found"/>
</p>

Now open "Keys" tab and generate new key as shown below. Copy its value - we will need it to setup release pipeline in the Azure DevOps:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease12.PNG?raw=true" alt="Image not found"/>
</p>


### Prepare GIT repository with policies files and script files ###

We will keep custom policies xml files in the GIT repository together with script I described below:
 
<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease13.PNG?raw=true" alt="Image not found"/>
</p>


Update-ADB2cPolicies.ps1 script is responsible for obtaining access token which is used to access Microsoft Graph API. Then script updates custom policies files in the Azure AD B2C tenant with obtained access token.

Update-ADB2cPolicies.ps1 script looks like below. 

**Important**

I modified the script which was originally published on [this website.](https://tech.xenit.se/querying-microsoft-graph-with-powershell-the-easy-way/)

Without this great content I would not be able to achieve policies automation.

Here I would like to also say thank you to [Kacper Mucha](https://www.linkedin.com/in/kacper-mucha-27559817/) who helped me a lot with script modifications.

```csharp
[CmdletBinding()]
param (
    $AdB2cAutomationAppId,
    $AdB2cAutomationAppSecret,
    $AdB2cAutomationTenantId,
    $CustomPolicyFileName,
    $CustomPolicyName
)

# Functions used to call Microsoft Graph API:
Function Get-MSGraphAuthToken{
    [cmdletbinding()]
    Param(
        [parameter(Mandatory=$true)]
        [pscredential]$credential,
        [parameter(Mandatory=$true)]
        [string]$tenantID
        )
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
        #Get token
        $AuthUri = "https://login.microsoftonline.com/$TenantID/oauth2/token"
        $Resource = 'graph.microsoft.com'
        $AuthBody = "grant_type=client_credentials&client_id=$($credential.UserName)&client_secret=$($credential.GetNetworkCredential().Password)&resource=https%3A%2F%2F$Resource%2F"
        
        $Response = Invoke-RestMethod -Method Post -Uri $AuthUri -Body $AuthBody
        If($Response.access_token){
            return $Response.access_token
        }
        Else{
            Throw "Authentication failed"
        }
    }
    
    Function Invoke-MSGraphQuery{
    
    [CmdletBinding(DefaultParametersetname="Default")]
    Param(
        [Parameter(Mandatory=$true,ParameterSetName='Default')]
        [Parameter(Mandatory=$true,ParameterSetName='Refresh')]
        [string]$URI,
    
        [Parameter(Mandatory=$false,ParameterSetName='Default')]
        [Parameter(Mandatory=$false,ParameterSetName='Refresh')]
        [string]$Body,
    
        [Parameter(Mandatory=$true,ParameterSetName='Default')]
        [Parameter(Mandatory=$true,ParameterSetName='Refresh')]
        [string]$token,
    
        [Parameter(Mandatory=$false,ParameterSetName='Default')]
        [Parameter(Mandatory=$false,ParameterSetName='Refresh')]
        [ValidateSet('GET','POST','PUT','PATCH','DELETE')]
        [string]$method = "GET",
            
        [Parameter(Mandatory=$false,ParameterSetName='Default')]
        [Parameter(Mandatory=$false,ParameterSetName='Refresh')]
        [switch]$recursive,
            
        [Parameter(Mandatory=$true,ParameterSetName='Refresh')]
        [switch]$tokenrefresh,
            
        [Parameter(Mandatory=$true,ParameterSetName='Refresh')]
        [pscredential]$credential,
            
        [Parameter(Mandatory=$true,ParameterSetName='Refresh')]
        [string]$tenantID
    )
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
        $authHeader = @{
            'Accept'= 'application/xml'
            'Content-Type'= 'application/xml'
            'Authorization'= $Token
        }
        [array]$returnvalue = $()
        Try{
            If($body){
                $Response = Invoke-RestMethod -Uri $URI -Headers $authHeader -Body $Body -Method $method -ErrorAction Stop
            }
            Else{
                $Response = Invoke-RestMethod -Uri $URI -Headers $authHeader -Method $method -ErrorAction Stop
            }
        }
        Catch{
            If(($Error[0].ErrorDetails.Message | ConvertFrom-Json -ErrorAction SilentlyContinue).error.Message -eq 'Access token has expired.' -and $tokenrefresh){
                $token =  Get-MSGraphAuthToken -credential $credential -tenantID $TenantID
    
                $authHeader = @{
                    'Accept'= 'application/xml'
                    'Content-Type'= 'application/xml'
                    'Authorization'=$Token
                }
                $returnvalue = $()
                If($body){
                    $Response = Invoke-RestMethod -Uri $URI –Headers $authHeader -Body $Body –Method $method -ErrorAction Stop
                }
                Else{
                    $Response = Invoke-RestMethod -Uri $uri –Headers $authHeader –Method $method
                }
            }
            Else{
                Throw $_
            }
        }
    
        $returnvalue += $Response
        If(-not $recursive -and $Response.'@odata.nextLink'){
            Write-Warning "Query contains more data, use recursive to get all!"
            Start-Sleep 1
        }
        ElseIf($recursive){
            If($PSCmdlet.ParameterSetName -eq 'default'){
                If($body){
                    $returnvalue += Invoke-MSGraphQuery -URI $Response.'@odata.nextLink' -token $token -body $body -method $method -recursive -ErrorAction SilentlyContinue
                }
                Else{
                    $returnvalue += Invoke-MSGraphQuery -URI $Response.'@odata.nextLink' -token $token -method $method -recursive -ErrorAction SilentlyContinue
                }
            }
            Else{
                If($body){
                    $returnvalue += Invoke-MSGraphQuery -URI $Response.'@odata.nextLink' -token $token -body $body -method $method -recursive -tokenrefresh -credential $credential -tenantID $TenantID -ErrorAction SilentlyContinue
                }
                Else{
                    $returnvalue += Invoke-MSGraphQuery -URI $Response.'@odata.nextLink' -token $token -method $method -recursive -tokenrefresh -credential $credential -tenantID $TenantID -ErrorAction SilentlyContinue
                }
            }
        }
        Return $returnvalue
    }


# Get custom policy file content

$customPolicyFileContent = Get-Content -Path "_ad-b2c-policies/policies/$CustomPolicyFileName.xml" -Raw

Write-Host $customPolicyFileContent


# Upload custom policy file to the Azure AD B2C:


$credential = New-Object System.Management.Automation.PSCredential($AdB2cAutomationAppId,(ConvertTo-SecureString $AdB2cAutomationAppSecret -AsPlainText -Force))
	
$token = Get-MSGraphAuthToken -credential $credential -tenantID $AdB2cAutomationTenantId

$URI = "https://graph.microsoft.com/beta/trustFramework/policies/B2C_1A_$CustomPolicyName/" + '$value'

Write-Host $URI

Invoke-MSGraphQuery -method PUT -URI $URI -Body $customPolicyFileContent -token $token
```


### Prepare release pipeline in Azure DevOps ###

From the "Pipelines" select "Release" and then click "New pipeline" button:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease14.PNG?raw=true" alt="Image not found"/>
</p>

Select "Empty job":

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease15.PNG?raw=true" alt="Image not found"/>
</p>

Type the name of the stage - in this case "Dev":

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease16.PNG?raw=true" alt="Image not found"/>
</p>

Configure artifacts by clicking "Add" button:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease17.PNG?raw=true" alt="Image not found"/>
</p>

Select GIT repo as source, then select project, repository and branch like presented below, then click "Add" button

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease18.PNG?raw=true" alt="Image not found"/>
</p>

Now select jobs section on the "Dev" stage:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease19.PNG?raw=true" alt="Image not found"/>
</p>

Set job configuration as presented below:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease20.PNG?raw=true" alt="Image not found"/>
</p>

Now add "PowerShell" tasks for each of custom policy file deploy.

Please note that in each task we have to set path to the PowerShell script located in the repository:

```csharp
$(System.DefaultWorkingDirectory)/_ad-b2c-policies/scripts/Update-ADB2cPolicies.ps1
```

We need to also pass arguments to the script:

1. CustomPolicyFileName - name of the policy file
2. CustomPolicyName - name of the policy
3. AdB2cAutomationAppId - app ID from the Azure portal
4. AdB2cAutomationAppSecret - app secret from the Azure portal (key)
5. AdB2cAutomationTenantId - tenant ID from the Azure portal

Below I present arguments for each task:


<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease26.PNG?raw=true" alt="Image not found"/>
</p>

```csharp
-CustomPolicyFileName $(TrustFrameworkBasePolicyFileName) -CustomPolicyName $(TrustFrameworkBasePolicyName) -AdB2cAutomationAppId $(AdB2cAutomationAppId) -AdB2cAutomationAppSecret $(AdB2cAutomationAppSecret) -AdB2cAutomationTenantId $(AdB2cAutomationTenantId)
```

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease27.PNG?raw=true" alt="Image not found"/>
</p>

```csharp
-CustomPolicyFileName $(TrustFrameworkExtensionsPolicyFileName) -CustomPolicyName $(TrustFrameworkExtensionsPolicyName) -AdB2cAutomationAppId $(AdB2cAutomationAppId) -AdB2cAutomationAppSecret $(AdB2cAutomationAppSecret) -AdB2cAutomationTenantId $(AdB2cAutomationTenantId)
```

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease28.PNG?raw=true" alt="Image not found"/>
</p>

```csharp
-CustomPolicyFileName $(SignUpOrSignInPolicyFileName) -CustomPolicyName $(SignUpOrSignInPolicyName) -AdB2cAutomationAppId $(AdB2cAutomationAppId) -AdB2cAutomationAppSecret $(AdB2cAutomationAppSecret) -AdB2cAutomationTenantId $(AdB2cAutomationTenantId)
```

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease29.PNG?raw=true" alt="Image not found"/>
</p>

```csharp
-CustomPolicyFileName $(ProfileEditPolicyFileName) -CustomPolicyName $(ProfileEditPolicyName) -AdB2cAutomationAppId $(AdB2cAutomationAppId) -AdB2cAutomationAppSecret $(AdB2cAutomationAppSecret) -AdB2cAutomationTenantId $(AdB2cAutomationTenantId)
```

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease29.PNG?raw=true" alt="Image not found"/>
</p>

```csharp
-CustomPolicyFileName $(PasswordResetPolicyFileName) -CustomPolicyName $(PasswordResetPolicyName) -AdB2cAutomationAppId $(AdB2cAutomationAppId) -AdB2cAutomationAppSecret $(AdB2cAutomationAppSecret) -AdB2cAutomationTenantId $(AdB2cAutomationTenantId)
```

**Add variables that will be injected as arguments to the script**

Now we have to add variables that will be used in the automation script:

| aa  |  aa |
|---|---|
| AdB2cAutomationAppId  |  ***
| AdB2cAutomationAppSecret  | ***
| AdB2cAutomationTenantId  |  ***
| PasswordResetPolicyFileName  | PasswordReset
| PasswordResetPolicyName  |  PasswordReset
| ProfileEditPolicyFileName  | ProfileEdit
| ProfileEditPolicyName  | ProfileEdit
| SignUpOrSignInPolicyFileName  | signup_signin
| SignUpOrSignInPolicyName  | signup_signin
| TrustFrameworkBasePolicyFileName  | TrustFrameworkBase
| TrustFrameworkBasePolicyName  | TrustFrameworkBase***
| TrustFrameworkExtensionsPolicyFileName  | TrustFrameworkExtensions
| TrustFrameworkExtensionsPolicyName  | TrustFrameworkExtensions

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease31.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease32.PNG?raw=true" alt="Image not found"/>
</p>


### Test release pipeline ###

Now it is time to test the release pipeline. First remove all policies in the AD B2C tab in the Azure portal (of course I assume you have them in the GIT repository):

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease34.PNG?raw=true" alt="Image not found"/>
</p>

Create a new release then:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease33.PNG?raw=true" alt="Image not found"/>
</p>

Wait and see the release pipeline status:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease36.PNG?raw=true" alt="Image not found"/>
</p>

Verify policies in the Azure portal. They should be deployed automatically:

<p align="center">
<img src="/images/devisland/article27/assets/B2cSeriesAutomaticRelease35.PNG?raw=true" alt="Image not found"/>
</p>


# Summary

In this article I presented how setup release pipeline in Azure DevOps to automatically update custom policies in the Azure AD B2C Identity Experience Framework.