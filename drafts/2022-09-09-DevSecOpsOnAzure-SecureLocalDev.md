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

Aaa


# Summary

Aaa


*If you like my articles, and find them helpful, please by me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>