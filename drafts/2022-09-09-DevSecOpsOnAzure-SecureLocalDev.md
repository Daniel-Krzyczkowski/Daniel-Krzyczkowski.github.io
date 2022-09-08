---
title: "DevSecOps on Azure - part2: Incorporate security at early stage on local dev machine"
excerpt: "This article presents approaches to secure development environment"
header:
  image: /images/devisland/article90/assets/devsecopsazure-local-dev01.png
---

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev01.png?raw=true" alt="DevSecOps on Azure - part2"/>
</p>

# Introduction

In my previous article I made an introduction to DevSecOps practices, and explained the concept. In this next article I would like to focus on development environment secuirty. Many of us focus on security in the CI/CD pipelines implemnentation, or Azure infrastructure. This is correct however vulnerabilities extends to the integrations which developers use in their local development environments as well. This is why it is important to utilize some secure DevOps practices when working with the code locally.

<p align="center">
<img src="/images/devisland/article90/assets/ddevsecopsazure-local-dev02.png?raw=true" alt="Image not found"/>
</p>

As we can see above, there are many threats related to development environment. Let's talk about how to incorporate security at early stage on local development machine.


# Storing secrets securely on local machine

One of the most popular mistake in DevOps world is commiting credentials to the source code repository accidentally. When developing applicaiton locally, we have to make sure that credentials can be stored securely and will be kept outside of the codebase commited to the repository.

There are many ways to avoid storing credentials in the code depending on what kind of language, and framework we use. When developing applications using Visual Studio and ASP .NET Core, we ca utilize [the Secret Manager](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-6.0&tabs=windows). With this tool we cab store sensitive data during the development of an ASP.NET Core project. The application secrets aren't checked into source control.

However it is very important to mention that Secret Manager  doesn't encrypt the stored secrets and shouldn't be treated as a trusted store. It's for development purposes only. The keys and values are stored in a JSON configuration file in the user profile directory. This is why when the development machine or process is compromised, environment variables can be accessed by untrusted parties.

<p align="center">
<img src="/images/devisland/article90/assets/ddevsecopsazure-local-dev03.png?raw=true" alt="Image not found"/>
</p>

As we can see, with the approach mentioned above, we can eliminate chance to commit any credentials to the source code repository but there is potential still security issue when someone will get access to the development machine. This is why cloud development environments are becoming more and more popular. They provide centralized control and templates in a cloud environment to securely configure, manage and grant access for developers. We are going to talk about Secure Cloud Development Environments later in this article.

# Security IDE plugins to scan the code as early as possible

Another good DevSecOps practice is to utilize security plugins for Integrated Development Environments (IDE). With such tools we can scan the code and detect potential vulnerabilities even before the code is commited to the source code repository. There are many different plugins and tools which can be used like:

* [Mend (WhiteSource previoslu)](https://www.mend.io/ide-integration/#form)
* [Sonar Lint](https://www.sonarsource.com/solutions/our-unique-approach/)
* [SonaType](https://marketplace.visualstudio.com/items?itemName=SonatypeCommunity.vscode-iq-plugin)
* [Snyk](https://marketplace.visualstudio.com/items?itemName=snyk-security.snyk-vulnerability-scanner)

In this article I would like to present how to utilize Snyk extension for Visual Studio Code to enable code scanning each time when new changes in the code are saved. Of course there are many more plugins and tools that can be used, however I found [Snyk](https://snyk.io/) the most comprechensive tool (it provides code scanning, license scanning, open source packages scanning, infrastructure scanning, containers scanning, and more).

### Enchance code security on local machine with Snyk

Aaaa


# Secure Cloud Development Environments

We can do our best to keep our local development machine secure however still there are some threats around. Security is one thing but also we can talk about power and configuration here. When working locally, each person in the team can have different type of cumputer, with different power. What is more - each developer can have own development environment setup. This can be problematic, and time consuming, especially when new person joins development team. Making sure that all required tools and plugins are correctly installed and configured is time consuming. This is why Cloud Development Environments can be really helpful and they can help improve security aspect of local development. Of course first thing that comes to mind is virtual machine, and remote access. However please remember that it can be challenging to maintain it, and provide proper (and secure access to all team members). This is why in this article I would like to talk about two helpful tools which can help keep development environments more secure - GitHub Codespaces, and Microsoft Dev Box.

## Secure dev environment with GitHub Codespaces

GitHub Codespaces provides cloud-powered development environments, which can be pre-configured and enabled for developers. With Codespaces we do not have to worry about setup of our local development environment. We can customize our project for GitHub Codespaces by committing configuration files to the repository (often known as Configuration-as-Code), which creates a repeatable codespace configuration for all users of your project. Just to clarify - a codespace is a development environment that's hosted in the cloud.

It is worth to mention that GitHub Codespaces are powered by Microsoft Azure cloud infrastructure.

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev04.png?raw=true" alt="Image not found"/>
</p>

As a developer we have access to editor which is Visual Studio Code. We can either use VS Code in the browser or use desktop version with [GitHub Codespaces plugin](https://code.visualstudio.com/docs/remote/codespaces) installed.

I recommend checking [official docuemntation](https://docs.github.com/en/codespaces/getting-started/quickstart) to read more. We can manage Codespaces for our organization. We can decide for which repositories Codespaces are available, restrict machine types, or set retention period.

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev06.PNG?raw=true" alt="Image not found"/>
</p>

Once we launch Codespaces, we have exactly the same experience as in the standard Visual Studio Code:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev05.PNG?raw=true" alt="Image not found"/>
</p>

It is worht to mention that when we work in a Codespaces, the environment is created using a development container, or dev container, hosted on a virtual machine. These are Docker containers that are specifically configured to provide a full-featured development environment. We can configure the dev container for a repository so that codespaces created for that repository give us a tailored development environment, complete with all the tools and runtimes we need to work on a specific project. The configuration files for a dev container are contained in a .devcontainer directory in our source code repository on GitHub.

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

Dev Box can be created directly from the Azure portal. Once it is ready, we can configure it and adjust to our needs:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev13.PNG?raw=true" alt="Image not found"/>
</p>

### Can we compare Dev Box to GitHub Actions?

There can be question asked - is DevBox the same as GitHub Codespaces? The anser is: not exactly. They are similar in their functionalities and targets but dirrenet when it comes to scope. Let me explain.

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev14.png?raw=true" alt="Image not found"/>
</p>

As we can see in the picture above, developer teams create and maintain dev box images with all the tools and dependencies their developers need to build and run their applications. Developer leads can instantly deploy the right size dev box for specific roles in a team anywhere in the world. By deploying dev boxes in the closest Azure region and connecting via the Azure Global Network dev teams ensure a smooth and responsive experience with gigabit connection speeds for developers around the world. This is similar to what GitHub Codespaces offers. However, Dev Box is focused on the organization needs, and enterprise environments.

Using Azure Active Directory groups, IT admins can grant access to sensitive source code and customer data for each project. With role-based permissions and custom network configurations, developer leads can give vendors limited access to the resources they need to contribute to the project—eliminating the need to ship hardware to short-term contractors and helping keep development more secure. Microsoft Dev Box builds on Windows 365, making it easy for IT administrators to manage dev boxes together with physical devices and Cloud PCs through Microsoft Intune and Microsoft Endpoint Manager. IT administrators can set conditional access policies to ensure users only access dev boxes from compliant devices while keeping dev boxes up to date using expedited quality updates to deploy zero-day patches across the organization and quickly isolate compromised devices.

### Sample configuration

Once we created Dev Box instance in the Azure portal, we can start configuring our development environments. Here is example of my *tmf-dev-box-definition-01* definition where I was able to inditace the target operating system for the Dev Box:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev15.png?raw=true" alt="Image not found"/>
</p>

What is really helpful when it comes to Dev Box is the fact that we can configure networking. With this feature we can connect our Dev Box to existing Azure Virtual Network, and securely develop, test, and deploy our solutions leveraging Microsoft backbone network infrastructure in Azure. This is another differentiator between GitHub Codespaces, and Microsoft Dev Box.

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev16.png?raw=true" alt="Image not found"/>
</p>

Once we configure networking (if needed obviously), we can setup new project. Projects enable us to manage team level settings and empower development teams to self-serve Dev Boxes:

<p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev17.png?raw=true" alt="Image not found"/>
</p>

 After we create the Dev Box, and configure it accordingly to our needs, we connect to it with a remote desktop (RD) session through a browser, or through a remote desktop app. It is great that we can use RBAC to control and grant access to Dev Box:

 <p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev18.png?raw=true" alt="Image not found"/>
</p>

As a developer, we can now sign in to [Dev Box portal](https://devbox.microsoft.com/), and create new development station for us:

 <p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev19.png?raw=true" alt="Image not found"/>
</p>

 <p align="center">
<img src="/images/devisland/article90/assets/devsecopsazure-local-dev20.png?raw=true" alt="Image not found"/>
</p>

This is just a short overview of course. I encourage you to read more about Microsoft Dev Box features in the [official ducumentation](https://docs.microsoft.com/en-us/azure/dev-box/overview-what-is-microsoft-dev-box).

# Summary

Aaa


*If you like my articles, and find them helpful, please by me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>