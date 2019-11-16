---
title: "DevSecOps With Microsoft Security Code Analysis Extension"
excerpt: "In this article would like to present Microsoft Security Code Analysis Extension for Azure DevOps to enable security scanning in the CI pipelines."
---

<p align="center">
<img src="/images/devisland/article28/assets/Msca.png?raw=true" alt="DevSecOps With Microsoft Security Code Analysis Extension"/>
</p>

Security scanning is one of the most important parts of DevSevOps practices. DevSecOps is about taking into account application and infrastructure security aspect. It also means automating security checks gates to keep the DevOps workflow healthy. There are many different tools available to apply security scanning in the DevOps cycle and one of them soon will be generally available - [Microsoft Security Code Analysis Extension](https://secdevtools.azurewebsites.net/). I had a pleasure to access preview version and make some tests to check what can be done with this extension for Azure DevOps from Microsoft.

# What is possible with Microsoft Security Code Analysis Extension?

The Microsoft Security Code Analysis extension makes the latest versions of important analysis tools readily available. It means that if there is an updated version of a tool, you don't need to download and install it manually - extension takes care of the updates.

The extension's build tasks hide the complexities of:

* Running security static-analysis tools.
* Processing the results from log files to create a summary report or break the build.


### Available tools

Below list contains tools that are currently available in the extension. More tools are expected to be added in the future.

<p align="center">
<img src="/images/devisland/article28/assets/Msca9.PNG?raw=true" alt="Image not found"/>
</p>


#### Anti-Malware Scanner

The Anti-Malware Scanner build task is now included in the Microsoft Security Code Analysis extension. This task must be run on a build agent that has Windows Defender already installed. For more information, see the [Windows Defender website](https://aka.ms/defender).

#### BinSkim

BinSkim is a Portable Executable (PE) lightweight scanner that validates compiler settings, linker settings, and other security-relevant characteristics of binary files. This build task provides a command-line wrapper around the binskim.exe console application. BinSkim is an open-source tool. For more information, see [BinSkim on GitHub](https://github.com/Microsoft/binskim).

#### Credential Scanner

Passwords and other secrets stored in source code are a significant problem. Credential Scanner is a proprietary static-analysis tool that helps solve this problem. The tool detects credentials, secrets, certificates, and other sensitive content in your source code and your build output.

#### Microsoft Security Risk Detection

Microsoft Security Risk Detection (MSRD) is a cloud-based service for fuzz testing. It identifies exploitable security bugs in software. This service requires a separate subscription and activation. For more information, see the [MSRD Developer Center](https://docs.microsoft.com/security-risk-detection/).

#### Roslyn Analyzers

Roslyn Analyzers is Microsoft's compiler-integrated tool for statically analyzing managed C# and Visual Basic code. For more information, see [Roslyn-based analyzers](https://docs.microsoft.com/dotnet/standard/analyzers/).

#### TSLint

TSLint is an extensible static-analysis tool that checks TypeScript code for readability, maintainability, and errors in functionality. It's widely supported by modern editors and build systems. You can customize it with your own lint rules, configurations, and formatters. TSLint is an open-source tool. For more information, see [TSLint on GitHub](https://github.com/palantir/tslint).


### Analysis results

Once above tasks are added the pipeline it is easy to access analysis results. The Microsoft Security Code Analysis extension also has three postprocessing tasks.

<p align="center">
<img src="/images/devisland/article28/assets/Msca8.PNG?raw=true" alt="Image not found"/>
</p>

#### Publish Security Analysis Logs

The Publish Security Analysis Logs build task preserves the log files of the security tools that are run during the build. You can read these logs for investigation and follow-up.

You can publish the log files to Azure Artifacts as a .zip file. You can also copy them to an accessible file share from your private build agent.

#### Security Report

The Security Report build task parses the log files. These files are created by the security tools that run during the build. The build task then creates a single summary report file. This file shows all issues found by the analysis tools.

You can configure this task to report results for specific tools or for all tools. You can also choose what issue level to report, like errors only or both errors and warnings.

#### Post-Analysis (build break)

With the Post-Analysis build task, you can inject a build break that purposely causes a build to fail. You inject a build break if one or more analysis tools report issues in the code.

You can configure this task to break the build for issues found by specific tools or all tools. You can also configure it based on the severity of issues found, such as errors or warnings.



# How to add Microsoft Security Code Analysis Extension task?

Adding Microsoft Security Code Analysis tools to the Azure DevOps CI pipeline is as simple as adding other new tasks. Once task is added it is possible to use its default configuration or customize it.


# Enable Microsoft Security Code Analysis Extension in Azure DevOps

At the moment of writing this article extension is in preview. If you would like to enable it for your Azure DevOps organization, please use this contact e-mail: sdt-vsts@microsoft.com. You can find details under [this link.](https://docs.microsoft.com/en-us/azure/security/develop/security-code-analysis-onboard#onboarding-the-microsoft-security-code-analysis-extension)

### Installation steps

First we have to install extension. To do it sign in to your Azure DevOps organization and select "Mange extensions" as presented below:

<p align="center">
<img src="/images/devisland/article28/assets/Msca1.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article28/assets/Msca2.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article28/assets/Msca3.PNG?raw=true" alt="Image not found"/>
</p>

Open "Shared" extensions section and click "Install" button:

<p align="center">
<img src="/images/devisland/article28/assets/Msca4.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article28/assets/Msca5.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article28/assets/Msca6.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article28/assets/Msca7.PNG?raw=true" alt="Image not found"/>
</p>

That's it! Now we are ready to start using Microsoft Security Code Analysis Extension.
Adding Microsoft Security Code Analysis tools to the Azure DevOps CI pipeline is as simple as adding other new tasks. Once task is added it is possible to use its default configuration or customize it.


### Example with scanning for ASP .NET Core Web API

Below I present integration of extension installed in my Azure DevOps organization and analysis result of tests.

I created sample ASP .NET Core Web API application and I wanted to check Credential Scanner task. To do it I hard-coded database connection string in the "appsettings.json" file. Below I present step by step how I integrated the task.

I used YAML build definition in this case.


#### Add Credential Scanner task

In the build pipeline definition I added "Credential Scanner" - it can be found with search box:

<p align="center">
<img src="/images/devisland/article28/assets/Msca11.PNG?raw=true" alt="Image not found"/>
</p>

I selected default setup for this task:

<p align="center">
<img src="/images/devisland/article28/assets/Msca12.PNG?raw=true" alt="Image not found"/>
</p>

#### Add Security Analysis Report task

Then I added task for creating a single summary report file basing on the scanning from the previous task (above).

<p align="center">
<img src="/images/devisland/article28/assets/Msca13.PNG?raw=true" alt="Image not found"/>
</p>

#### Add Publish Security Analysis Logs task

This task enables publishing the log files to Azure Artifacts as a .zip file. With this task I was able to download HTML site report about the scanning analysis.

<p align="center">
<img src="/images/devisland/article28/assets/Msca15.PNG?raw=true" alt="Image not found"/>
</p>

#### Add Post-Analysis (build break) task

This task enables breaking the build if one or more analysis tools reported issues in the code.

<p align="center">
<img src="/images/devisland/article28/assets/Msca14.PNG?raw=true" alt="Image not found"/>
</p>

#### Queue the build and check the result

Once I queued the build - it was braked by the "Post-Analysis" task because credentials were find:

<p align="center">
<img src="/images/devisland/article28/assets/Msca16.PNG?raw=true" alt="Image not found"/>
</p>

When I opened details about this task there was a clear information where credentials were detected:

<p align="center">
<img src="/images/devisland/article28/assets/Msca17.PNG?raw=true" alt="Image not found"/>
</p>

#### Download and open full report

It is great that ready report is prepared and it can be downloaded:

<p align="center">
<img src="/images/devisland/article28/assets/Msca18.PNG?raw=true" alt="Image not found"/>
</p>

Once I downloaded it I opened HTML file with report:

<p align="center">
<img src="/images/devisland/article28/assets/Msca19.PNG?raw=true" alt="Image not found"/>
</p>

As you can see clear information was provided where credentials were used in the source code:

*ConfigFile {{Code}}See appsettings.json line 9 for the code resulting in match {{Info}}Found password, symmetric key or storage credential in source file.*

What is more! When I clicked "View" button new tab was opened with the place in the source code repository where credentials were detected:

<p align="center">
<img src="/images/devisland/article28/assets/Msca20.PNG?raw=true" alt="Image not found"/>
</p>

This is the whole YAML build pipeline definition I have:

```csharp
# ASP.NET Core
# Build and test ASP.NET Core projects targeting .NET Core.
# Add steps that run tests, create a NuGet package, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- master

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'

steps:

- task: DotNetCoreCLI@2
  displayName: 'Restore project dependencies'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: MSBuild@1
  inputs:
    solution: '**/*.sln'

- task: CredScan@2

- task: RoslynAnalyzers@2
  inputs:
    userProvideBuildInfo: 'auto'

- task: SdtReport@1
  inputs:
    TsvFile: false
    AllTools: false
    BinSkim: false
    CredScan: true
    MSRD: false
    RoslynAnalyzers: true
    RoslynAnalyzersBreakOn: 'WarningAbove'
    TSLint: false
    ToolLogsNotFoundAction: 'Standard'

- task: PublishSecurityAnalysisLogs@2
  inputs:
    ArtifactName: 'CodeAnalysisLogs'
    ArtifactType: 'Container'
    AllTools: true
    ToolLogsNotFoundAction: 'Standard'

- task: PostAnalysis@1
  inputs:
    AllTools: false
    BinSkim: false
    CredScan: true
    RoslynAnalyzers: true
    RoslynAnalyzersBreakOn: 'Error'
    TSLint: false
    ToolLogsNotFoundAction: 'Standard'

- task: DotNetCoreCLI@2
  displayName: 'dotnet publish'
  inputs:
    command: 'publish'
    publishWebProjects: true
    arguments: '--configuration $(BuildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    zipAfterPublish: True

- task: PublishBuildArtifacts@1
  displayName: 'publish artifacts'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'

```

You will notice that there is also "Roslyn Analyzers" task which I added to do some static code analysis.


# Summary

In this article I presented Microsoft Security Code Analysis Extension which enables security scanning in the CI pipelines. This extension is important part of DevSecOps flow and definitely will be worth trying once it is generally available. If you would like to read more about Security Engineering I recommend to visit [Microsoft  Security Development Lifecycle (SDL) website.](https://www.microsoft.com/en-us/securityengineering/sdl/practices)