---
title: "Azure Hints Series - Secure DevOps for Azure Infrastructure"
excerpt: "This article presents how to scan Azure infrastructure security issues and send them to Microsoft Sentinel to generate alerts"
header:
  image: /images/devisland/article86/assets/azure-hints-01-devsecops-for-infra.png
---

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra.png?raw=true" alt="Azure Hints Series - Secure DevOps for Azure Infrastructure"/>
</p>

# Introduction

Infrastructure as Code enables DevOps teams to create cloud environments quickly and store infrastructure code in the source code repository. This approach also enables the possibility to validate and test the infrastructure code and configuration to prevent common deployment and security issues.

In this first article from the Azure Hints Series, I would like to show how to keep infrastructure code secure and implement the process of constant monitoring for Azure infrastructure templates using Microsoft Sentinel, Azure DevOps, and [Checkmark's KICS](https://kics.io) - open-source solution for static code analysis of Infrastructure as Code.

# Process explanation

The below diagram explains the process implemented using Azure DevOps pipelines:

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-02.png?raw=true" alt="Image not found"/>
</p>

Let me explain the steps:

1. First there is a new pull request created
2. Azure Pipeline is triggered
3. Bicep files are converted to ARM (JSON) - this is required as of now KICS does not support Bicep directly
4. ARM files are scanned with KICS and a security report is generated
5. Security scanning report is sent to Azure Log Analytics Workspace using Log Analytics REST API
6. Analyze logs with Microsoft Sentinel Analytics Rule
7. Send an email with an alert if needed
8. Stop executing pipeline if "HIGH" severity issues were detected


# End result

Below I provided some screenshots to show the end result.


### Azure DevOps pipeline

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-03.png?raw=true" alt="Image not found"/>
</p>

### Scanning result

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-04.png?raw=true" alt="Image not found"/>
</p>

### Microsoft Sentinel dashboard

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-05.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-06.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-07.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-08.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-11.png?raw=true" alt="Image not found"/>
</p>


# Implementation

Now it is time to talk a little bit about implementation.

## Convert Bicep to ARM task

As I mentioned before, first we have to convert Bicep to ARM (JSON) files. Here is the task responsible for this operation:


```yaml
      - task: AzureCLI@2
        displayName: 'Convert Bicep template to ARM'
        inputs:
          azureSubscription: $(azureSubscriptionConnectionName)
          scriptType: bash
          scriptLocation: inlineScript
          inlineScript: |
            az bicep build --file $(bicepFilePathForSecurityScan) --outdir $(Build.ArtifactStagingDirectory)
```

We have to provide *azureSubscriptionConnectionName* parameter together with the path to Bicep file that should be converted (the main one) using *bicepFilePathForSecurityScan* parameter.

## Scan infrastructure code with Checkmarx's KICS

The next step is to scan infrastructure (ARM) code with KICS tool. Here is the code to achieve it:

```yaml
    - script: |
        /app/bin/kics scan --ci -p $(System.ArtifactsDirectory)/arm -o ${PWD} --report-formats json,sarif --ignore-on-exit results
        cat results.json
        TOTAL_SEVERITY_COUNTER=`grep '"total_counter"':' ' results.json | awk {'print $2'}`
        export SEVERITY_COUNTER_HIGH=`grep '"HIGH"':' ' results.json | awk {'print $2'} | sed 's/.$//'`
        SEVERITY_COUNTER_MEDIUM=`grep '"INFO"':' ' results.json | awk {'print $2'} | sed 's/.$//'`
        SEVERITY_COUNTER_LOW=`grep '"LOW"':' ' results.json | awk {'print $2'} | sed 's/.$//'`
        SEVERITY_COUNTER_INFO=`grep '"MEDIUM"':' ' results.json | awk {'print $2'} | sed 's/.$//'`
        echo "TOTAL SEVERITY COUNTER: $TOTAL_SEVERITY_COUNTER"
        echo "##vso[task.setvariable variable=highSeverityIssuesCounter;isOutput=true]$SEVERITY_COUNTER_HIGH"
      displayName: 'Scan infrastructure code'
      name: infraCodeSecurityScan

    # scan results should be visible in the SARIF viewer tab of the build - SCANS tab
    - task: PublishBuildArtifacts@1
      displayName: 'Generate infrastructure scanning report in SARIF'
      inputs:
        pathToPublish: $(System.DefaultWorkingDirectory)/results.sarif
        artifactName: CodeAnalysisLogs

    - task: PublishBuildArtifacts@1
      displayName: 'Generate infrastructure scanning report in JSON'
      inputs:
        pathToPublish: $(System.DefaultWorkingDirectory)/results.json
        artifactName: CodeAnalysisJson
```
First, code is analyzed with KICS, then SARIF raport is generated, and report in JSON format. We will use the last one to send logs to Azure Log Analytics.

One more important notice - KICS requires using *container* job in the Azure DevOps pipeline:


```yaml
  - job: Scan_With_Kics
    dependsOn: Conver_Bicep_To_ARM
    condition: succeeded()
    displayName: 'Scan infrastructure code'
    pool:
      vmImage: "ubuntu-latest"
    container: checkmarx/kics:debian
    steps:
    - task: DownloadPipelineArtifact@2
      displayName: 'Download ARM files'
      inputs:
          artifactName: 'armFiles'
          downloadPath: '$(System.ArtifactsDirectory)/arm'
          
    - template: ../tasks/scan.infrastructure.with.kics.task.yml
```

Here is the structure of generated JSON report:


```json
{
  "kics_version": "v1.5.8",
  "files_scanned": 1,
  "lines_scanned": 1246,
  "files_parsed": 1,
  "lines_parsed": 1246,
  "files_failed_to_scan": 0,
  "queries_total": 42,
  "queries_failed_to_execute": 0,
  "queries_failed_to_compute_similarity_id": 0,
  "scan_id": "console",
  "severity_counters": {
    "HIGH": 4,
    "INFO": 0,
    "LOW": 1,
    "MEDIUM": 0,
    "TRACE": 0
  },
  "total_counter": 5,
  "total_bom_resources": 0,
  "start": "2022-05-25T03:51:29.661322371Z",
  "end": "2022-05-25T03:51:32.356792905Z",
  "paths": [
    "/__w/1/a/arm"
  ],
  "queries": [
    {
      "query_name": "Key Vault Not Recoverable",
      "query_id": "7c25f361-7c66-44bf-9b69-022acd5eb4bd",
      "query_url": "https://docs.microsoft.com/en-us/azure/templates/microsoft.keyvault/2019-09-01/vaults?tabs=json#vaultproperties-object",
      "severity": "HIGH",
      "platform": "AzureResourceManager",
      "category": "Backup",
      "description": "Key Vault should have 'enableSoftDelete' and 'enablePurgeProtection' set to true",
      "description_id": "8e3ca202",
      "cis_description_id": "CIS Security - CIS Microsoft Azure Foundations Benchmark v1.3.1 - Rule 8.4",
      "cis_description_title": "Ensure the key vault is recoverable",
      "cis_description_text": "The key vault contains object keys, secrets, and certificates. Accidental unavailability of a key vault can cause immediate data loss or loss of security functions (authentication, validation, verification, non-repudiation, etc.) supported by the key vault objects. It is recommended the key vault be made recoverable by enabling the \"Do Not Purge\" and \"Soft Delete\" functions. This is in order to prevent the loss of encrypted data including storage accounts, SQL databases, and/or dependent services provided by key vault objects (Keys, Secrets, Certificates) etc., as may happen in the case of accidental deletion by a user or from disruptive activity by a malicious user.\nThere could be scenarios where users accidentally run delete/purge commands on key vault or attacker/malicious user does it deliberately to cause disruption. Deleting or purging a key vault leads to immediate data loss as keys encrypting data and secrets/certificates allowing access/services will become non-accessible.There are 2 key vault properties that plays role in permanent unavailability of a key vault. enableSoftDelete : Setting this parameter to true for a key vault ensures that even if key vault is deleted, Key vault itself or its objects remain recoverable for next 90days. In this span of 90 days either key vault/objects can be recovered or purged (permanent deletion). If no action is taken, after 90 days key vault and its objects will be purged. enablePurgeProtection : enableSoftDelete only ensures that key vault is not deleted permanently and will be recoverable for 90 days from date of deletion. However, there are chances that the key vault and/or its objects are accidentally purged and hence will not be recoverable. Setting enablePurgeProtection to \"true\" ensures that the key vault and its objects cannot be purged. Enabling both the parameters on key vaults ensures that key vaults and their objects cannot be deleted/purged permanently.",
      "files": [
        {
          "file_name": "../a/arm/main.json",
          "similarity_id": "784fbfd587834006aa604663859099bbbbb63f6102e7ddf23ad260015b5e4680",
          "line": 580,
          "issue_type": "MissingAttribute",
          "search_key": "resources.properties.template.resources.name={{[parameters('keyVaultName')]}}.properties",
          "search_line": 0,
          "search_value": "",
          "expected_value": "resource with type 'Microsoft.KeyVault/vaults' has 'enableSoftDelete' property defined",
          "actual_value": "resource with type 'Microsoft.KeyVault/vaults' doesn't have 'enableSoftDelete' property defined"
        },
        {
          "file_name": "../a/arm/main.json",
          "similarity_id": "784fbfd587834006aa604663859099bbbbb63f6102e7ddf23ad260015b5e4680",
          "line": 580,
          "issue_type": "MissingAttribute",
          "search_key": "resources.properties.template.resources.name={{[parameters('keyVaultName')]}}.properties",
          "search_line": 0,
          "search_value": "",
          "expected_value": "resource with type 'Microsoft.KeyVault/vaults' has 'enablePurgeProtection' property defined",
          "actual_value": "resource with type 'Microsoft.KeyVault/vaults' doesn't have 'enablePurgeProtection' property defined"
        }
      ]
    },
        ...
  ]
}
```

## Send scanning result to Azure Log Analytics

Once the report is generated in a JSON file, we can send it directly to Azure Log Analytics. Here is the code to achieve it:

```yaml
parameters:
  - name: scriptFilePath
    type: string
  - name: workspaceId
    type: string
  - name: key
    type: string
  - name: patToJsonFile
    type: string
  - name: logType
    type: string

steps:
  - task: PowerShell@2
    displayName: 'Send logs to Azure Log Analytics'
    inputs:
      filePath: ${{parameters.scriptFilePath}}
      arguments: >-
        -WorkspaceId ${{parameters.workspaceId}}
        -Key ${{parameters.key}}
        -PatToJsonFile ${{parameters.patToJsonFile}}
        -LogType "${{parameters.logType}}"
```

I use the PowerShell script to send the logs. Here is the full script code:

```powershell
[Cmdletbinding()]
Param(
    [Parameter(Mandatory = $true)][string]$WorkspaceId,
    [Parameter(Mandatory = $true)][string]$Key,
    [Parameter(Mandatory = $true)][string]$PatToJsonFile,
    [Parameter(Mandatory = $true)][string]$LogType
)

# Create the function to create the authorization signature
Function Build-Signature ($customerId, $sharedKey, $date, $contentLength, $method, $contentType, $resource)
{
    $xHeaders = "x-ms-date:" + $date
    $stringToHash = $method + "`n" + $contentLength + "`n" + $contentType + "`n" + $xHeaders + "`n" + $resource

    $bytesToHash = [Text.Encoding]::UTF8.GetBytes($stringToHash)
    $keyBytes = [Convert]::FromBase64String($sharedKey)

    $sha256 = New-Object System.Security.Cryptography.HMACSHA256
    $sha256.Key = $keyBytes
    $calculatedHash = $sha256.ComputeHash($bytesToHash)
    $encodedHash = [Convert]::ToBase64String($calculatedHash)
    $authorization = 'SharedKey {0}:{1}' -f $customerId,$encodedHash
    return $authorization
}

# Create the function to create and post the request
Function Post-LogAnalyticsData($customerId, $sharedKey, $body, $logType)
{
    $method = "POST"
    $contentType = "application/json"
    $resource = "/api/logs"
    $rfc1123date = [DateTime]::UtcNow.ToString("r")
    $contentLength = $body.Length
    $signature = Build-Signature `
        -customerId $customerId `
        -sharedKey $sharedKey `
        -date $rfc1123date `
        -contentLength $contentLength `
        -method $method `
        -contentType $contentType `
        -resource $resource
    $uri = "https://" + $customerId + ".ods.opinsights.azure.com" + $resource + "?api-version=2016-04-01"

    $headers = @{
        "Authorization" = $signature;
        "Log-Type" = $logType;
        "x-ms-date" = $rfc1123date;
        "time-generated-field" = $TimeStampField;
    }

    $response = Invoke-WebRequest -Uri $uri -Method $method -ContentType $contentType -Headers $headers -Body $body -UseBasicParsing
    return $response.StatusCode

}

$jsonFileContent = Get-Content $PatToJsonFile | Out-String

# Submit the data to the API endpoint
$response = Post-LogAnalyticsData -customerId $WorkspaceId -sharedKey $Key -body ([System.Text.Encoding]::UTF8.GetBytes($jsonFileContent)) -logType $LogType 

Write-Host "Successfully sent logs to Azure Log Analytics. Response code: $response.StatusCode"
```

In the above script, we have to provide the below parameters:

1. WorkspaceId - ID of our Log Analytics Workspace
2. Key - Primary Key of our Log Analytics Workspace
3. PatToJsonFile - Path to JSON file with scanning result
4. LogType - name of the logs, in my case I named it *InfrastructureSecurityScan*


## Validate HIGH severity results and stop the pipeline

Once logs are sent, I want to break the pipeline execution if there are HIGH severity issues with the infrastructure code. To do it I have created the below task:

```yaml
  - script: |
      echo "SEVERITY COUNTER: $(highSeverityIssuesCounter)"
      SEVERITY_COUNTER_HIGH=$(highSeverityIssuesCounter)
      if [ "$SEVERITY_COUNTER_HIGH" -ge "1" ]; then
        echo "Please review all $SEVERITY_COUNTER issues with infrastructure code" && exit 1;
      fi
    displayName: 'Validate scanning result'
```

Here is the structure of stages, jobs, and tasks in the Azure DevOps for this process:

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-09.png?raw=true" alt="Image not found"/>
</p>

I also created dedicated variable group to keep all required variables:

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-10.png?raw=true" alt="Image not found"/>
</p>


## Microsoft Sentinel Analytics Rule

Once the JSON report is sent to Azure Log Analytics, we have to create Analytic Rule in Microsoft Sentinel to query the logs and react with an email alert when there are issues with infrastructure code detected. Here is the query that verifies the logs generated 30 minutes ago. This analytic rule can be executed in a specific time period so it is important to properly set this time range value parameter. For tests my Analytic Rule is executed every 5 minutes:

```text
InfrastructureSecurityScan_CL
| where TimeGenerated > ago(30m) 
| mvexpand parsejson(queries_s)
| extend queryName=queries_s["query_name"]
| extend severity=queries_s["severity"]
| extend description=queries_s["description"]
| extend fileNames=queries_s["files"]
| mvexpand parsejson(fileNames)
| extend fileName=fileNames["file_name"]
| extend actualValue=fileNames["actual_value"]
| extend expectedValue=fileNames["expected_value"]
| project queryName, severity, description, fileName, actualValue, expectedValue
```

<p align="center">
<img src="/images/devisland/article86/assets/azure-hints-01-devsecops-for-infra-11.png?raw=true" alt="Image not found"/>
</p>

# Scanning infrastructure code on local machine

With *shift left* approach, we want to solve all bugs and security issues at the earlier step, before they will be discovered on the production environment. Infrastructure scanning can be also executed on the local machine using Docker. [Here](https://docs.kics.io/latest/getting-started/) you can read more about how to set up it.

# Summary

In this article, I explained how to set up Azure infrastructure code scanning and auditing using Azure DevOps, Microsoft Sentinel, and Checkmarx's KICS. It is worth mentioning that when it comes to KICS, it can be used to scan other platforms like *CDK*, or *Terraform*.
