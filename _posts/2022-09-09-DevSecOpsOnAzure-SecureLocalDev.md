---
title: "DevSecOps on Azure - part2: Incorporate security at an early stage"
excerpt: "This article presents approaches to secure development environment"
header:
  image: /images/devisland/article90/assets/devsecopsazure-local-dev01.png
---

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev01.png?raw=true" alt="DevSecOps on Azure - part2"/>
</p>

# Introduction

In my [previous article](https://techmindfactory.com/DevSecOpsOnAzure-Introduction/) I made an introduction to DevSecOps practices and explained the concept. In this next article, I would like to focus on development environment security. Many of us focus on security in the CI/CD pipeline implementation, or Azure infrastructure. This is correct however vulnerabilities extend to the integrations which developers use in their local development environments as well. This is why it is important to utilize some secure DevOps practices when working with the code locally.

<p align="center">
<img src="/images/devisland/article90/assets/ddevsecopsazure-local-dev02.png?raw=true" alt="Image not found"/>
</p>

As we can see above, there are many threats related to the development environment. Let's talk about how to incorporate security at an early stage on the local development machine.


# Storing secrets securely on the local machine

One of the most popular mistakes in the DevOps world is committing credentials to the source code repository accidentally. When developing an application locally, we have to make sure that credentials can be stored securely and will be kept outside of the codebase committed to the repository.

There are many ways to avoid storing credentials in the code depending on what kind of language, and framework we use. When developing applications using Visual Studio and ASP .NET Core, we can utilize [the Secret Manager](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-6.0&tabs=windows). With this tool we can store sensitive data during the development of an ASP.NET Core project. The application secrets aren't checked into source control.

However, it is very important to mention that Secret Manager doesn't encrypt the stored secrets and shouldn't be treated as a trusted store. It's for development purposes only. The keys and values are stored in a JSON configuration file in the user profile directory. This is why when the development machine or process is compromised, environment variables can be accessed by untrusted parties.

<p align="center">
<img src="/images/devisland/article90/assets/ddevsecopsazure-local-dev03.png?raw=true" alt="Image not found"/>
</p>

As we can see, with the approach mentioned above, we can eliminate the chance to commit any credentials to the source code repository but there is potential still for security issues when someone will get access to the development machine. This is why cloud development environments are becoming more and more popular. They provide centralized control and templates in a cloud environment to securely configure, manage and grant access to developers. We are going to talk about Secure Cloud Development Environments later in this article.

# Security IDE plugins to scan the code as early as possible

Another good DevSecOps practice is to utilize security plugins for Integrated Development Environments (IDE). With such tools, we can scan the code and detect potential vulnerabilities even before the code is committed to the source code repository. There are many different plugins and tools which can be used like:

* [Mend (WhiteSource previoslu)](https://www.mend.io/ide-integration/#form)
* [Sonar Lint](https://www.sonarsource.com/solutions/our-unique-approach/)
* [SonaType](https://marketplace.visualstudio.com/items?itemName=SonatypeCommunity.vscode-iq-plugin)
* [Snyk](https://marketplace.visualstudio.com/items?itemName=snyk-security.snyk-vulnerability-scanner)

In this article, I would like to present how to utilize the Snyk extension for Visual Studio Code to enable code scanning each time when new changes in the code are saved. Of course, there are many more plugins and tools that can be used, however, I found [Snyk](https://snyk.io/) the most comprehensive tool (it provides code scanning, license scanning, open source packages scanning, infrastructure scanning, containers scanning, and more). It offers [a free plan](https://snyk.io/plans/).

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev22.png?raw=true" alt="Image not found"/>
</p>

### Enhance code security on the local machine with Snyk

We can keep our code secure from the early stages. Snyk provides extensions for bot, [Visual Studio](https://docs.snyk.io/ide-tools/visual-studio-extension), and [Visual Studio Code](https://docs.snyk.io/ide-tools/visual-studio-code-extension-for-snyk-code) IDEs. Once the plugin is installed, we can start using it. It is great because each time we save changes in our project, Snyk scans our whole codebase. There is also a dedicated section where we can see all scan results. We can also manually start scanning:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev21.PNG?raw=true" alt="Image not found"/>
</p>

With such an approach, we can easily get information about code quality and security before even committing anything to the source code repository.


### Enhance code security in CI/CD

Once we verify our code on the local machine, we can push it to the source code repository in GitHub, or Azure DevOps. Snyk supports both tools so we can integrate scanning in GitHub Actions, and Azure DevOps pipelines. After scanning, we can access the report directly from GitHub or Azure DevOps:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev23.PNG?raw=true" alt="Image not found"/>
</p>

Here is the Azure DevOps Snyk task:

```YAML
  - task: SnykSecurityScan@1
    displayName: "Apply security scan"
    inputs:
      serviceConnectionEndpoint: "snyk-connection"
      testType: "app"
      severityThreshold: "medium"
      monitorWhen: "always"
      failOnIssues: true
      additionalArguments: "--file=src/TMF.sln"
```

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev24.PNG?raw=true" alt="Image not found"/>
</p>

Here are GitHub Snyk Actions:

```YAML
    - name: Run Snyk to check for vulnerabilities
      uses: snyk/actions/dotnet@master
      continue-on-error: true # To make sure that SARIF upload gets called
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --sarif-file-output=snyk.sarif --file=./src/web-game/Globomantics.sln --severity-threshold=medium


    - name: Upload result to GitHub Code Scanning
      uses: github/codeql-action/upload-sarif@v1
      with:
        sarif_file: snyk.sarif
```

Snyk also offers license scanning with a higher plan enabled. It can be really helpful to verify the licenses of OSS libraries we use in our commercial projects. Lack of knowledge about specific license types can lead to serious legal issues.

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev25.PNG?raw=true" alt="Image not found"/>
</p>

This is just a short overview of the course. I encourage you to read more about Snyk features in the [official ducumentation](https://docs.snyk.io/).


# Secure Cloud Development Environments

We can do our best to keep our local development machine secure however still there are some threats around. Security is one thing but also we can talk about power and configuration here. When working locally, each person in the team can have different type of computer, with different power. What is more - each developer can have their development environment setup. This can be problematic, and time-consuming, especially when a new person joins the development team. Making sure that all required tools and plugins are correctly installed and configured is time-consuming. This is why Cloud Development Environments can be helpful and they can help improve the security aspect of local development. Of course the first thing that comes to mind is a virtual machine and remote access. However please remember that it can be challenging to maintain it, and provide proper (and secure access to all team members). This is why in this article I would like to talk about two helpful tools which can help keep development environments more secure - GitHub Codespaces, and Microsoft Dev Box.

## Secure dev environment with GitHub Codespaces

GitHub Codespaces provides cloud-powered development environments, which can be pre-configured and enabled for developers. With Codespaces we do not have to worry about the setup of our local development environment. We can customize our project for GitHub Codespaces by committing configuration files to the repository (often known as Configuration-as-Code), which creates a repeatable codespace configuration for all users of your project. Just to clarify - a codespace is a development environment that's hosted in the cloud.

It is worth mentioning that GitHub Codespaces are powered by Microsoft Azure cloud infrastructure.

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev04.png?raw=true" alt="Image not found"/>
</p>

As a developer, we have access to editor which is Visual Studio Code. We can either use VS Code in the browser or use the desktop version with [GitHub Codespaces plugin](https://code.visualstudio.com/docs/remote/codespaces) installed.

I recommend checking [official documentation](https://docs.github.com/en/codespaces/getting-started/quickstart) to read more. We can manage Codespaces for our organization. We can decide for which repositories Codespaces are available, restrict machine types, or set retention period.

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev06.PNG?raw=true" alt="Image not found"/>
</p>

Once we launch Codespaces, we have the same experience as in the standard Visual Studio Code:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev05.PNG?raw=true" alt="Image not found"/>
</p>

It is worth mentioning that when we work in a Codespaces, the environment is created using a development container, or dev container, hosted on a virtual machine. These are Docker containers that are specifically configured to provide a full-featured development environment. We can configure the dev container for a repository so that codespaces created for that repository give us a tailored development environment, complete with all the tools and runtimes we need to work on a specific project. The configuration files for a dev container are contained in a .devcontainer directory in our source code repository on GitHub.

When we create new Codespaces, we can decide which branch, region, and machine size we want to use:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev07.PNG?raw=true" alt="Image not found"/>
</p>

Then cloud development environment is configured for us:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev08.PNG?raw=true" alt="Image not found"/>
</p>

With command palete we can select container configuration definition:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev09.PNG?raw=true" alt="Image not found"/>
</p>


We can also specify additional features that should be installed, like Azure CLI, or PowerShell. Once we do it, .devcontainer.json file lands in our repository together with DockerFile:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev10.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev11.PNG?raw=true" alt="Image not found"/>
</p>

We can also launch VS Code locally also but powered by Codespaces. If we want to finish development, we can stop our Codespace:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev12.PNG?raw=true" alt="Image not found"/>
</p>

This is just a short overview of course. I encourage you to read more about Codespaces features in the [official ducumentation](https://docs.github.com/en/codespaces/overview).

## Secure dev environment with Microsoft Dev Box

Microsoft Dev Box (in preview at the moment of writing this article) is a new cloud service that provides developers with secure, ready-to-code developer workstations. Development teams preconfigure Dev Boxes for specific projects and tasks, enabling devs to get started quickly with an environment that’s ready to build and run their app in minutes.

Dev Box can be created directly from the Azure portal. Once it is ready, we can configure it and adjust it to our needs:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev13.PNG?raw=true" alt="Image not found"/>
</p>

### Can we compare Dev Box to GitHub Actions?

There can be a question asked - is DevBox the same as GitHub Codespaces? The answer is: not exactly. They are similar in their functionalities and targets but dirrenet when it comes to scope. Let me explain.

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev14.png?raw=true" alt="Image not found"/>
</p>

As we can see in the picture above, developer teams create and maintain dev box images with all the tools and dependencies their developers need to build and run their applications. Developer leads can instantly deploy the right size dev box for specific roles in a team anywhere in the world. By deploying dev boxes in the closest Azure region and connecting via the Azure Global Network dev teams ensure a smooth and responsive experience with gigabit connection speeds for developers around the world. This is similar to what GitHub Codespaces offers. However, Dev Box is focused on the organization's needs and enterprise environments.

Using Azure Active Directory groups, IT admins can grant access to sensitive source code and customer data for each project. With role-based permissions and custom network configurations, developer leads can give vendors limited access to the resources they need to contribute to the project—eliminating the need to ship hardware to short-term contractors and helping keep development more secure. Microsoft Dev Box builds on Windows 365, making it easy for IT administrators to manage dev boxes together with physical devices and Cloud PCs through Microsoft Intune and Microsoft Endpoint Manager. IT administrators can set conditional access policies to ensure users only access dev boxes from compliant devices while keeping dev boxes up to date using expedited quality updates to deploy zero-day patches across the organization and quickly isolate compromised devices.

### Sample configuration

Once we created Dev Box instance in the Azure portal, we can start configuring our development environments. Here is an example of my *tmf-dev-box-definition-01* definition where I was able to indicate the target operating system for the Dev Box:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev15.png?raw=true" alt="Image not found"/>
</p>

What is helpful when it comes to Dev Box is the fact that we can configure networking. With this feature, we can connect our Dev Box to the existing Azure Virtual Network, and securely develop, test, and deploy our solutions leveraging Microsoft backbone network infrastructure in Azure. This is another differentiator between GitHub Codespaces and Microsoft Dev Box.

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev16.png?raw=true" alt="Image not found"/>
</p>

Once we configure networking (if needed obviously), we can set up a new project. Projects enable us to manage team-level settings and empower development teams to self-serve Dev Boxes:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev17.png?raw=true" alt="Image not found"/>
</p>

 After we create the Dev Box, and configure it accordingly to our needs, we connect to it with a remote desktop (RD) session through a browser, or a remote desktop app. It is great that we can use RBAC to control and grant access to Dev Box:

 <p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev18.png?raw=true" alt="Image not found"/>
</p>

As a developer, we can now sign in to [Dev Box portal](https://devbox.microsoft.com/), and create a new development station for us:

 <p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev19.png?raw=true" alt="Image not found"/>
</p>

 <p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev20.png?raw=true" alt="Image not found"/>
</p>

This is just a short overview of the course. I encourage you to read more about Microsoft Dev Box features in the [official ducumentation](https://docs.microsoft.com/en-us/azure/dev-box/overview-what-is-microsoft-dev-box).

# Summary

In this article, I explained how to incorporate security at an early stage on a local dev machine. We also went through the cloud development environments which are the future in my opinion because we can have pre-configured developer environments with enhanced security. In the next article, we will talk about container security.


*If you like my articles, and find them helpful, please buy me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>