---
title: "DevSecOps on Azure - part4: Enhance security in Continuous Integration pipeline"
excerpt: "This article presents practices to enhance security in Continuous Integration pipeline"
header:
  image: /images/devisland/article92/assets/devsecopsazure-enhance-ci-security-01.png
---

<p align="center">
<img src="/images/devisland/article92/assets/devsecopsazure-enhance-ci-security-01.png?raw=true" alt="DevSecOps on Azure - part4"/>
</p>

# Introduction

This is the next article from the series called *DevSecOps practices for Azure cloud workloads*. In this article I would like to talk about security aspect in the Continuous Integration pipelines. We can do our best to secure Azure environment where our workloads are running but it is also important to enhance security at the earlier stage. This is why in this article we will discover how to enhance security when architecting Continuous Integration pipeline. I will use GitHub, and Azure DevOps tools to show sample implementations.

# Introduction to SBOM

Every software consists of many different components. Software Bill of Materials (SBOM) is the list of ingredients that make up software components, providing software transparency so organizations have insight into their supply chain dependencies. An SBOM is a formal, machine-readable inventory of software components and dependencies, information about those components, and their hierarchical relationships. SBOM was mentioned as a formal requirement in the [*Executive Order on Improving the Nation’s Cybersecurity document*](https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/). There are many different details and statements, and one of them requires *providing a purchaser a Software Bill of Materials (SBOM) for each product directly or by publishing it on a public website*. Here is also very important part:

*The term “Software Bill of Materials” or “SBOM” means a formal record containing the details and supply chain relationships of various components used in building software.  Software developers and vendors often create products by assembling existing open source and commercial software components.  The SBOM enumerates these components in a product.  It is analogous to a list of ingredients on food packaging.  An SBOM is useful to those who develop or manufacture software, those who select or purchase software, and those who operate software.  Developers often use available open source and third-party software components to create a product; an SBOM allows the builder to make sure those components are up to date and to respond quickly to new vulnerabilities.  Buyers can use an SBOM to perform vulnerability or license analysis, both of which can be used to evaluate risk in a product.  Those who operate software can use SBOMs to quickly and easily determine whether they are at potential risk of a newly discovered vulnerability.   A widely used, machine-readable SBOM format allows for greater benefits through automation and tool integration.  The SBOMs gain greater value when collectively stored in a repository that can be easily queried by other applications and systems.  Understanding the supply chain of software, obtaining an SBOM, and using it to analyze known vulnerabilities are crucial in managing risk.*

This is why it is very important to be able to generate SBOM report. Before we will discover how to do it, let's talk a little bit about the structure.

## The Minimum Elements for an SBOM

The core of an SBOM is a consistent, uniform structure that captures and presents information
used to understand the components that make up software. The goal of these fields is to enable sufficient identification of these components to track them across the software supply chain and map them to other beneficial sources of data, such as vulnerability databases or license databases. This baseline component information includes:

<p align="center">
<img src="/images/devisland/article92/assets/devsecopsazure-enhance-ci-security-02.PNG?raw=true" alt="Image not found"/>
</p>

While a single standard may offer simplicity and efficiency, multiple data formats exist in the
ecosystem and are being used to generate and consume SBOMs like:

* Software Package Data eXchange [(SPDX)](https://spdx.dev/)
* [CycloneDX13](https://github.com/CycloneDX)
* Software Identification [(SWID)](https://nvd.nist.gov/products/swid)


## Open-source SBOM tool from Microsoft

There are many tools which can be used to generate SBOM report. In this article I would like to show how to use [SBOM Tool](https://github.com/microsoft/sbom-tool) - open source tool from Microsoft. SBOM tool is a general purpose, enterprise-proven, build-time SBOM generator. It works across platforms including Windows, Linux, and Mac, and uses the standard Software Package Data Exchange (SPDX) format mentioned above.

It can be easily integrated into and auto-detects NPM, NuGet, PyPI, CocoaPods, Maven, Golang, Rust Crates, RubyGems, Linux packages within containers, Gradle, Ivy, GitHub public repositories, and more.

SBOM report generated by this tool contains four main sections based on the SPDX specification:

1. **Document creation information** - General information about the SBOM document, such as software name, SPDX license, SPDX version, who created the document, when it was created.
2. **Files section** - A list of files that compose the piece of software. Each file has some properties including the hashes of its content (SHA-1, SHA-256).
3. **Packages section** - A list of packages used when building the software. Each package has additional properties such as name, version, supplier, hashes (SHA-1, SHA-256) and a Package URL (purl) software identifier.
4. **Relationships section** - A list of relationships between the different elements of the SBOM, such as files and packages.

I took above information from the official announcement made in this [*Microsoft open sources its software bill of materials (SBOM) generation tool*](https://devblogs.microsoft.com/engineering-at-microsoft/microsoft-open-sources-software-bill-of-materials-sbom-generation-tool/) blog post.

It is worth to mention that above tool can be integrated with Azure DevOps, and GitHub.


## Generate SBOMs with SBOM Tool 

In this section we will see how to generate SBOM report on the local machine. First, we have to download [MS SBOM Tool](https://github.com/microsoft/sbom-tool/releases). Once tool is downloaded, we can use it using command line interface (CMD). For the demo purpose I created new *Class Library* project, and added two nuget packages to it: [Dapper](https://www.nuget.org/packages/Dapper/), and [Microsoft Identity Web](https://www.nuget.org/packages/Microsoft.Identity.Web).


Next, I moved the SBOM Tool exe file to the folder where my project is located. Here is the command I used to generate SBOM report:

```CMD
sbom-tool generate -b C:\Users\Daniel\Desktop\demo-tmf-package\TMF\sbom-report -ps DanielK -bc C:\Users\Daniel\Desktop\demo-tmf-package\TMF\TMF.MyOSSLibrary -pn myTMFPackage -pv 1.0.0 -nsb https://techmindfactory.com/TMF
```

* **BuildDropPath (-b)** - The root folder of the drop directory for which the manifest file will be generated.
* **PackageSupplier (-ps)** - Supplier of the package that this SBOM represents.
* **BuildComponentPath (-bc)** - The folder containing the build components and packages.
* **PackageName (-pn)** - The name of the package this SBOM represents. If this is not provided, we will try to infer this name from the build that generated this package.
* **PackageVersion (-pv)** - The version of the package this SBOM represents. If this is not provided, we will try to infer the version from the build that generated this package, if that also fails, the SBOM generation fails.
* **NamespaceUriBase (-nsb)** - The base path of the SBOM namespace URI.

We can find list of all available paremeters [in the official documentation here](https://github.com/microsoft/sbom-tool/blob/main/docs/sbom-tool-arguments.md).

Here is the information provided in the console after executing the command:

<p align="center">
<img src="/images/devisland/article92/assets/devsecopsazure-enhance-ci-security-03.PNG?raw=true" alt="Image not found"/>
</p>

We can see above that NuGet packages were detected in my project. There is also information about the packages I directly referenced in my project. This is the SBOM report generated in the SPDX format:

```JSON
{
  "files": [],
  "packages": [
    {
      "name": "Microsoft.AspNetCore.DataProtection",
      "SPDXID": "SPDXRef-Package-E3723B6678C04E67E73EC2476C2547059AC85B447FF43D195026C6586B0AD2A5",
      "downloadLocation": "NOASSERTION",
      "filesAnalyzed": false,
      "licenseConcluded": "NOASSERTION",
      "licenseInfoFromFiles": [
        "NOASSERTION"
      ],
      "licenseDeclared": "NOASSERTION",
      "copyrightText": "NOASSERTION",
      "versionInfo": "5.0.8",
      "externalRefs": [
        {
          "referenceCategory": "PACKAGE-MANAGER",
          "referenceType": "purl",
          "referenceLocator": "pkg:nuget/Microsoft.AspNetCore.DataProtection%405.0.8"
        }
      ],
      "supplier": "NOASSERTION"
    },
    {
      "name": "Microsoft.Extensions.DependencyInjection.Abstractions",
      "SPDXID": "SPDXRef-Package-FE12D2A286D8EC366C3A77906C3D32888C2FDFB4609B60F36C2159748DCFFF53",
      "downloadLocation": "NOASSERTION",
      "filesAnalyzed": false,
      "licenseConcluded": "NOASSERTION",
      "licenseInfoFromFiles": [
        "NOASSERTION"
      ],
      "licenseDeclared": "NOASSERTION",
      "copyrightText": "NOASSERTION",
      "versionInfo": "5.0.0",
      "externalRefs": [
        {
          "referenceCategory": "PACKAGE-MANAGER",
          "referenceType": "purl",
          "referenceLocator": "pkg:nuget/Microsoft.Extensions.DependencyInjection.Abstractions%405.0.0"
        }
      ],
      "supplier": "NOASSERTION"
    },
    {
      "name": "Microsoft.IdentityModel.JsonWebTokens",
      "SPDXID": "SPDXRef-Package-960EFB2FD9AD5C0686431D828FDD93A0061F91ECC33AC3DF0CAFFE9F815DE41B",
      "downloadLocation": "NOASSERTION",
      "filesAnalyzed": false,
      "licenseConcluded": "NOASSERTION",
      "licenseInfoFromFiles": [
        "NOASSERTION"
      ],
      "licenseDeclared": "NOASSERTION",
      "copyrightText": "NOASSERTION",
      "versionInfo": "6.23.1",
      "externalRefs": [
        {
          "referenceCategory": "PACKAGE-MANAGER",
          "referenceType": "purl",
          "referenceLocator": "pkg:nuget/Microsoft.IdentityModel.JsonWebTokens%406.23.1"
        }
      ],
      "supplier": "NOASSERTION"
    },
    ...REMOVED FOR BREVITY...
  ],
  "externalDocumentRefs": [],
  "relationships": [
    {
      "relationshipType": "DEPENDS_ON",
      "relatedSpdxElement": "SPDXRef-Package-EE0742950FBD84D832594518492E8ACD511A6DF57408D54450151DE8C6BB8A23",
      "spdxElementId": "SPDXRef-RootPackage"
    },
    ...REMOVED FOR BREVITY...
  ],
  "spdxVersion": "SPDX-2.2",
  "dataLicense": "CC0-1.0",
  "SPDXID": "SPDXRef-DOCUMENT",
  "name": "myTMFPackage 1.0.0",
  "documentNamespace": "https://techmindfactory.com/TMF/myTMFPackage/1.0.0/E6YXRbOoqkajQfABNmDhTQ",
  "creationInfo": {
    "created": "2022-09-20T06:06:54Z",
    "creators": [
      "Organization: DanielK",
      "Tool: Microsoft.SBOMTool-0.2.2"
    ]
  },
  "documentDescribes": [
    "SPDXRef-RootPackage"
  ]
}
```

### Generate SBOMs in Azure DevOps pipeline

Generating SBOM report locally is always helpful but having it automated, and integrated into CI pipeline can be also beneficial. Below we can see example how to extend Azure DevOps pipeline to generate SBOM report.

```YAML
pool:
  vmImage: ubuntu-latest

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '6.x'

# First, we have to build our project:
- script: |
    dotnet build $(Build.SourcesDirectory)/TMF.CoreLibrary.csproj --output $(Build.ArtifactStagingDirectory)
  displayName: 'Build the project'

# Next, we have to use CURL to get sbom-tool and run scunning:
- script: |
    curl -Lo $(Agent.TempDirectory)/sbom-tool https://github.com/microsoft/sbom-tool/releases/latest/download/sbom-tool-linux-x64
    chmod +x $(Agent.TempDirectory)/sbom-tool
    $(Agent.TempDirectory)/sbom-tool generate -b $(Build.ArtifactStagingDirectory) -bc $(Build.SourcesDirectory) -pn Test -pv 1.0.0 -ps TechMindFactory -nsb https://techmindfactory.com/TMF -V Verbose
  displayName: Generate SBOM report

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'
```

Report is available under artifacts:

<p align="center">
<img src="/images/devisland/article92/assets/devsecopsazure-enhance-ci-security-04.png?raw=true" alt="Image not found"/>
</p>


### Generate SBOMs in GitHub Actions

Below we can see example how to extend GitHub Actions to generate SBOM report.

```YAML
name: Tech Mind Factory SBOM

on: 
  workflow_dispatch:

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Setup .NET
      uses: actions/setup-dotnet@v2
      with:
        dotnet-version: 6.0.x
    # First, we have to build our project:
    - name: Build
      run: dotnet build Sample.sln --output buildOutput
    # Next, we have to use CURL to get sbom-tool and run scunning:  
    - name: Generate SBOM
      run: |
        curl -Lo $RUNNER_TEMP/sbom-tool https://github.com/microsoft/sbom-tool/releases/latest/download/sbom-tool-linux-x64
        chmod +x $RUNNER_TEMP/sbom-tool
        $RUNNER_TEMP/sbom-tool generate -b ./buildOutput -bc . -pn Test -pv 1.0.0 -ps TechMindFactory -nsb https://techmindfactory.com/TMF -V Verbose

    - name: Upload a Build Artifact
      uses: actions/upload-artifact@v3.1.0
      with:
        path: buildOutput
```

With such SBOM report generated, as software supplier we can provide transparent information about the components that make up our software.


# Software Composition Analysis to detect vulnerabilities

Now when we know what Software Bill of Materials (SBOM) is, we can talk about Software Composition Analysis (SCA). With Software Composition Analysis we cab inspect package managers, manifest files, source code, binary files, container images. The identified open source is compiled into **a Software Bill of Materials (SBOM)**, which is then compared against a variety of databases, including the National Vulnerability Database (NVD). Open source components are becoming major building blocks in software, and SCA tools help keeping track their version, and security state. SCA tools can also compare SBOMs against databases to discover licenses associated with the code and analyze overall code quality.

There are many good SCA tools. One of them is [Snyk Open Source](https://snyk.io/product/open-source-security-management/).


# Static Application Security Testing (SAST)

Aaa


# Securely storing and using parameters in the Continuous Integration pipelines

Aaa


# Summary

In this article...


*If you like my articles, and find them helpful, please buy me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
