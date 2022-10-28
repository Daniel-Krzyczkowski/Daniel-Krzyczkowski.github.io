---
title: "DevSecOps on Azure - part9: Improve the security of Azure environment and DevOps platforms"
excerpt: "This article presents how to monitor the security posture of Azure cloud environments and DevOps platforms"
header:
  image: /images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-01.png
---

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-01.png?raw=true" alt="DevSecOps on Azure - part9"/>
</p>

# Introduction

This is the next article from the series called *DevSecOps practices for Azure cloud workloads*. security monitoring with Microsoft Defender for Cloud and DevOps. Keeping application solutions on Azure secure requires constant security monitoring and evaluation of threats from source code up to running workloads. In this article, I would like to focus on the Azure cloud environment and security of DevOps platforms - GitHub and Azure DevOps.


# Understand Microsoft Defender for cloud

Defender for Cloud is a solution for cloud security posture management (CSPM) and cloud workload protection (CWP). It is used to secure native Azure resources, on-premises resources, and other cloud providers resources (Amazon AWS and Google GCP). Defender for Cloud finds weak spots across Azure services configuration, helps strengthen the overall security posture of the cloud environment, and can protect application workloads from evolving threats.

Defender for Cloud addresses the most urgent security challenges:

1. Limited security skills and specialists
2. Increased rate of sophisticated attacks
3. Dynamically changing workloads


## Defend cloud environment in Azure with Microsoft Defender features

First of all, it is worth knowing that not all Defender for Cloud features are paid.

### Free, core features of Microsoft Defender for Cloud

There are two core, free features of Microsoft Defender for Cloud:

1. Security recommendations - hints that should be followed to make Azure workloads more secure.
2. Azure Security Benchmark Policies - Azure Policy Initiative with security policies for Azure resources.

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-02.PNG?raw=true" alt="Image not found"/>
</p>

Recommendations are tailored to the particular security concerns found in the Azure workloads. Defender for Cloud not only provides information about security posture but also specific instructions for how to improve it. Recommendations help reduce the attack surface across each Azure resource. Free pricing tier is enabled on all current Azure subscriptions when Defender for Cloud is opened in the Azure portal for the first time or enabled through the API.

Azure Security Benchmark Azure Policy Initiative is automatically assigned to all Defender for Cloud registered subscriptions. The built-in initiative contains only Audit policies to not break current solutions and configurations. Only recommendations are displayed. Defender for Cloud policies are visible in the Azure portal Policies section. Security compliance can be verified on specific Azure resources.

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-03.PNG?raw=true" alt="Image not found"/>
</p>

#### Use Azure Policies together with Microsoft Defender for cloud 

As mentioned above, Azure Security Benchmark policies are automatically assigned to subscriptions connected to Defender for Cloud. Many different policies help improve security posture. Some of them provide information on how to manually fix the security issues:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-04.PNG?raw=true" alt="Image not found"/>
</p>

Some of those policies provide a direct "quick fix" option to fix the issue with security posture of the Azure resource:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-05.PNG?raw=true" alt="Image not found"/>
</p>

As mentioned before, the built-in Azure Policy Initiative contains only Audit policies to not break current solutions and configurations.

Defender for Cloud continually assesses Azure resources and subscriptions for security issues. It then aggregates all the findings into a single secure score.
It shows the general, current security situation. 
The higher the score, the lower the identified risk level.

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-06.png?raw=true" alt="Image not found"/>
</p>


### Standard (paid) Defender for Cloud tier features

To take advantage of advanced security management and threat detection capabilities, a standard, paid tier has to be enabled. Below there are some features of the paid plan:

1. Vulnerability assessment for virtual machines, container registries, and SQL resources
2. Multicloud security (AWS, GCP)
3. Hybrid (on-premises) security 
4. Compliance tracking with a range of standards (PCI, SOC)

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-07.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-08.PNG?raw=true" alt="Image not found"/>
</p>

Here is an example of security protection for containers- images in the Azure Container Registry are constantly scanned and if the vulnerability is found, we are informed about this fact together with recommendations on how to mitigate it:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-09.PNG?raw=true" alt="Image not found"/>
</p>

## Security alerts and incidents

Defender for Cloud and Defender for Cloud plans will generate alerts when threats in our cloud, hybrid, or on-premises environment are detected. Each alert provides details of affected resources, issues, and remediation recommendations. Defender for Cloud leverages the [MITRE Attack Matrix](https://attack.mitre.org/matrices/enterprise/) to associate alerts with their perceived intent, helping formalize security domain knowledge.

Defender for Cloud assigns a severity to alerts to help us prioritize how we attend to each alert. Severity is based on how confident Defender for Cloud is in the:

* Finding/analytic used to issue the alert
* Confidence level that there was malicious intent behind the activity that led to the alert

A security incident is a collection of related alerts. Incidents provide you with a single view of an attack and its related alerts so that you can quickly understand actions an attacker took, and the resources affected.

Here is an example alert that will be sent to our mailbox when a new threat is detected:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-10.PNG?raw=true" alt="Image not found"/>
</p>

Under *Security alerts* tab in the Azure Portal we can access all alerts and read about details:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-11.PNG?raw=true" alt="Image not found"/>
</p>

Next, we can take actions to mitigate security risk and solve the problem:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-12.PNG?raw=true" alt="Image not found"/>
</p>


# Defender for DevOps

At the moment of writing this article, Defender for DevOps is in preview, it was announced during Microsoft Ignite 2022. I would like to write about it and some really interesting features when it comes to secure DevOps and the security of DevOps platforms - GitHub and Azure DevOps.

Until now it was not so obvious how to monitor source code repositories, CI/CD pipelines, and general security and access in DevOps platforms like Azure DevOps or GitHub. This is where Defender for DevOps can help. We can connect Azure DevOps or GitHub (and more platforms in the future) to Defender for DevOps and monitor holistic security posture of them.

Key capabilities in Defender for DevOps include:

* Unified visibility into DevOps security posture
* Strengthen cloud resource configurations throughout the development lifecycle
* Prioritize remediation of critical issues in source code

We can add GitHub and Azure DevOps environments, customize DevOps workbooks to show desired metrics, view our guides and give feedback, and configure pull request annotations. We can review findings like:

1. Code scanning findings - number of code vulnerabilities and misconfigurations identified in the connected repositories.
2. OSS vulnerabilities – number of open source dependency vulnerabilities identified in the repositories (only available for GitHub for now).
3. Exposed secrets - number of secrets identified in the repositories.
4. Pull request annotation status - to show whether Pull Request annotations are enabled for the repository (available only for Azure DevOps now).
5. Repositories - lists with connected repositories from GitHub and Azure DevOps platforms.


## Connect DevOps platforms to Defender for DevOps

There is well-written documentation on how to connect DevOps platforms with Defender for DevOps:

1. [Connect GitHub to Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/quickstart-onboard-github)
2. [Connect Azure DevOps to Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/quickstart-onboard-devops)

### Microsoft Security DevOps extension

During the Microsoft Ignite, there was the announcement that [GitHub Advanced Security will be available for Azure DevOps](https://devblogs.microsoft.com/devops/integrate-security-into-your-developer-workflow-with-github-advanced-security-for-azure-devops/#github-advanced-security). It will provide secret scanning, dependency scanning, and code scanning. At the moment of writing this article, this feature is in private preview.

However, when I was searching in the Defender for Cloud documentation, I found new extension that can be used with Azure DevOps and GitHub called *Microsoft Security DevOps*. This extension provides great capabilities to increase the security posture of our source code repositories together with code quality and security. Microsoft Security DevOps is data-driven with portable configurations that enable deterministic execution across multiple environments.

With Microsoft Security DevOps we can:

1. Detect secrets in our source code repository
2. We can scan Azure infrastructure source code (like ARM, Bicep, Terraform)
3. Container image scanning
4. Source code quality scanning

With this extension, all security incidents are reported back to the Microsoft Defender for DevOps (from Azure DevOps and GitHub).

Here is an example of secret scanning in Azure DevOps:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-14.PNG?raw=true" alt="Image not found"/>
</p>

Here is the security report in the Defender for DevOps dashboard:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-13.PNG?raw=true" alt="Image not found"/>
</p>

It is really helpful because Security Teams receive detailed information where the secret was detected and they can take actions to inform developers about this fact:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-17.PNG?raw=true" alt="Image not found"/>
</p>

Now there is a question - what about GitHub Advanced Security for Azure DevOps and GitHub? 

I am not sure yet but in my personal opinion it will be possible to utilize both options:

1. Microsoft Security DevOps extension with Azure DevOps and GitHub
2. GitHub Advanced Security with Azure DevOps and GitHub

The case is that the two above can report security issues to the Defender for DevOps. Now let me provide some more details.

#### Microsoft Security DevOps Azure DevOps extension

The extension is available for free to be installed from [Azure DevOps marketplace](https://marketplace.visualstudio.com/items?itemName=ms-securitydevops.microsoft-security-devops-azdevops). Once it is installed in our Azure DevOps organization, we can start using it.

Here is an example job from my sample pipeline to detect secrets:

```YAML
jobs:
  - job: 'Build'
    displayName: "Build Web API"
    pool:
      vmImage: 'windows-latest'
    steps:

    - task: UseDotNet@2
      displayName: 'Install .NET 6 SDK'
      inputs:
        packageType: 'sdk'
        version: 6.0.x
    # Here we scan the code with Microsoft Security DevOps:
    - task: MicrosoftSecurityDevOps@1
      displayName: 'Microsoft Security DevOps scan'
      inputs:
        categories: 'secrets'
    
    - task: DotNetCoreCLI@2
      displayName: Restore NuGet packages
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
      
    - task: DotNetCoreCLI@2
      displayName: Build project
      inputs:
        command: 'build'
        projects: '**/*.csproj'

    - task: DotNetCoreCLI@2
      displayName: Publish project
      inputs:
        command: publish
        arguments: '--configuration Release --output $(Build.ArtifactStagingDirectory)'
        projects: '**/*.csproj'
        zipAfterPublish: true
      
    - task: PublishBuildArtifacts@1
      displayName: Publish package ready for deployment
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'
```

Currently, it works only with *vmImage: 'windows-latest'* agents - this is important because if you use Linux agents, no scanning output will be available.

Microsoft Security DevOps uses the following Open Source tools:

| Name | Language | License |
|--|--|--|
| [Bandit](https://github.com/PyCQA/bandit) | Python | [Apache License 2.0](https://github.com/PyCQA/bandit/blob/master/LICENSE) |
| [BinSkim](https://github.com/Microsoft/binskim) | Binary--Windows, ELF | [MIT License](https://github.com/microsoft/binskim/blob/main/LICENSE) |
| [ESlint](https://github.com/eslint/eslint) | JavaScript | [MIT License](https://github.com/eslint/eslint/blob/main/LICENSE) |
| [Credscan](detect-credential-leaks.md) | Credential Scanner (also known as CredScan) is a tool developed and maintained by Microsoft to identify credential leaks such as those in source code and configuration files <br> common types: default passwords, SQL connection strings, Certificates with private keys | Not Open Source |
| [Template Analyzer](https://github.com/Azure/template-analyzer) | ARM template, Bicep file | [MIT License](https://github.com/Azure/template-analyzer/blob/main/LICENSE.txt) |
| [Terrascan](https://github.com/accurics/terrascan) | Terraform (HCL2), Kubernetes (JSON/YAML), Helm v3, Kustomize, Dockerfiles, Cloud Formation | [Apache License 2.0](https://github.com/accurics/terrascan/blob/master/LICENSE) |
| [Trivy](https://github.com/aquasecurity/trivy) | container images, file systems, git repositories | [Apache License 2.0](https://github.com/aquasecurity/trivy/blob/main/LICENSE) |

Please note that the extension utilizes *Credential Scanner* which originally was part of [*Microsoft Security Code Analysis (MSCA) extension*](https://learn.microsoft.com/en-us/azure/security/develop/security-code-analysis-overview) which will be retired on December 31, 2022.

With *Credential Scanner* we can detect secrets in our source code during the build process and we do not have to utilize GitHub Advanced Security features. Here is the link to [official documentation to read more](https://learn.microsoft.com/en-us/azure/defender-for-cloud/azure-devops-extension).


#### Microsoft Security DevOps GitHub action

Action is available for free to be used. We have to add action to our workflow like in the example below:

```YAML
name: MSDO windows-latest
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  sample:

    # MSDO runs on windows-latest and ubuntu-latest.
    # macos-latest supporting coming soon
    runs-on: windows-latest

  steps:
  - uses: actions/checkout@v2

  - uses: actions/setup-dotnet@v1
    with:
      dotnet-version: |
        5.0.x
        6.0.x

    # Run analyzers
    - name: Run Microsoft Security DevOps Analysis
      uses: microsoft/security-devops-action@preview
      id: msdo

  # Upload alerts to the Security tab
  - name: Upload alerts to Security tab
    uses: github/codeql-action/upload-sarif@v1
    with:
      sarif_file: ${{ steps.msdo.outputs.sarifFile }}
```

Once the scanning is completed, we can access it under *Security > Code scanning alerts* tab. Here is the link to [official documentation to read more](https://learn.microsoft.com/en-us/azure/defender-for-cloud/github-action).

It is worth mentioning that this action does not utilize *Credential Scanner* mentioned above for Azure DevOps. The reason why is probably the fact that GitHub natively supports secret scanning and detection for public repositories and GitHub Advanced Security for private repositories.


### Infrastructure scanning example

As mentioned before, we can also scan Azure infrastructure source code. Once we enable it, the report is generated in the separate tab:

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-15.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article97/assets/devsecopsazure-improve-sec-with-defender-16.PNG?raw=true" alt="Image not found"/>
</p>

This is a comprehensive extension for DevSecOps! I am also waiting for dedicated connector for Microsoft Sentinel to connect Defender for DevOps.

# Summary

In this article, we discussed how to improve the security posture of the application environment in the Azure cloud with Defender for Cloud and how to monitor DevOps platforms (GitHub and Azure DevOps) with Defender for DevOps. In the next, last article we will see how to detect and respond to security events in Azure with Microsoft Sentinel.


*If you like my articles, and find them helpful, please buy me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
