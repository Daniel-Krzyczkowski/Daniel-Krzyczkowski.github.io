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

## Setup secure dev environment with GitHub Codespaces

Aaa

## Setup secure dev environment with Microsoft Dev Box

Aaa


# Summary

Aaa

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-10.PNG?raw=true" alt="Image not found"/>
</p>



*If you like my articles, and find them helpful, please by me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>