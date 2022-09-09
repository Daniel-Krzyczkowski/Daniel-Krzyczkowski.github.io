---
title: "Introduction to DevSecOps on Azure"
excerpt: "This article contains an introduction to DevSecOps practices for Azure cloud workloads"
header:
  image: /images/devisland/article89/assets/devsecopsazure-intro-01.png
---

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-01.png?raw=true" alt="Introduction to DevSecOps on Azure"/>
</p>

# Introduction

DevOps practices are crucial when building and maintaining cloud solutions. DevOps provides many different benefits like:

- Faster, better features delivery
- Faster issue resolution and reduced complexity
- More stable operating environments
- Greater automation
- Greater visibility into system outcomes

However many times we forget about security in this whole "DevOps loop" which is also very important. This is why it is beneficial to extend the DevOps approach with security practices to end up with Secure DevOps, or DevSecOps. Before we jump into practices, technical details, and many other things, it is good to explain what DevSecOps is.


# Explanation of DevSecOps

DevSecOps applies innovation security by integrating security processes and tools into the DevOps development process.
As many organizations prepare to adopt DevOps practices, it is crucial to adapt security and other governance processes to our planned approach. This approach mitigates security risk but doesn't decrease the value of rapid release cycles.

Integrating security late in the process is expensive and hard to fix. This is why we should *Shift security left* in the timeline to integrate it into the envisioning, design, implementation, and operation phases of the DevOps cycle.

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-02.png?raw=true" alt="Image not found"/>
</p>

Many organizations look to software development to gain a competitive advantage through innovation.
Protecting this innovation requires that organizations address potential security weaknesses and attacks in both the development process and the infrastructure hosting the applications. This approach is related to both worlds: cloud and on-premises.

Attackers might exploit weaknesses in the development process (attackers might find a weakness in the implementation of the design, for example, code doesn't validate input and allows common attacks like SQL injection) but also in the IT infrastructure (attackers might also conduct a multistage attack that uses stolen credentials or malware to access development infrastructure from other parts of the environment).

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-03.png?raw=true" alt="Image not found"/>
</p>

To summarize, when we talk about DevSecOps, these three parts should be kept in mind:

1. **Dev** - Application development aligned with business and user needs, which are often rapidly evolving.
2. **Sec** - Application must be resilient to attacks from rapidly evolving attackers and take advantage of innovations in security defenses
3. **Ops** - Application must be reliable and perform efficiently



# Review of DevSecOps practices on Azure 

DevOps practices are quite clear when it comes to solutions implementation in the Microsoft Azure cloud. We can plan our solution using Azure DevOps Boards, we can configure CI/CD with Azure Pipelines or GitHub Actions, and we can monitor the solution using Azure Monitor, and Application Insights. DevSecOps is the integration of security into every stage of the DevOps lifecycle from idea inception through envisioning, architectural design, iterative application development, and operations. Let's discuss some of the DevSecOps practices for Azure cloud solutions to make them more secure from the idea to production deployment.

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-04.png?raw=true" alt="Image not found"/>
</p>

The above diagram presents some DevSecOps practices for Azure cloud workloads. Of course, there can be more elements but I decided to keep the most crucial ones, which should be always included when we build the secure solution on Azure cloud.

### Plan and develop

Security can be incorporated at a very early stage of the development process. In the past, security checks were done later in the process (often just before deployment to the production environment). Introducing security into this part of the development process should focus on:

- Threat modeling - to view the application through the lens of a potential attacker
- IDE security plug-ins - to apply static analysis checking within an integrated development environment (IDE)
- Branch protection rules - to avoid direct commits to specific branches in the source code repository
- Peer review - to improve code quality, and detect potential security issues



### Secure CI/CD

Once the code is pushed to the source code repository, it is important to have the right security checks incorporated into CI/CD process. Important security controls at this stage include:

- Static Application Security Testing (SAST) - to detect potential security risks in the developer's written code
- Software Composition Analysis (SCA) - to know what components (often open source packages) are included in the application code, keep versions up to date, and detect vulnerabilities in these components
- Cloud configuration validation - to make sure that the environments we deploy applications into are also secure, so in Azure, we can Azure Policy to prevent the creation of certain workload types or configuration items
- Infrastructure scanning - to scan infrastructure code, and detect potential security issues with its configuration
- Secure pipelines - DevOps teams must ensure they implement the proper security controls for the pipelines to avoid security incidents and unwanted access
- Dynamic Application Security Testing (DAST) - to look for any vulnerabilities in the infrastructure or application configuration, which might create weaknesses that attackers can exploit. One of the most popular security approaches is penetration testing or pen testing

### Operate

Once our solution is available on the production environment, there is still space for incorporating DevSevOps practices. At this stage in the process, it's time to focus on the cloud infrastructure and overall application.

- Solution monitoring - to keep an eye on potential issues with the Azure infrastructure or application execution
- Threat intelligence - to detect and alert any anomalous events or configurations that require investigation and potential remediation, with tools like Microsoft Sentinel, or Microsoft Defender for Cloud
- Penetration tests - to check for any vulnerabilities in the infrastructure or application configuration, which might create weaknesses that attackers can exploit.

A key element of DevSecOps is data-driven, event-driven processes. These processes help teams identify, evaluate, and respond to potential risks. With tools like Azure DevOps or GitHub, it is much easier to implement all crucial DevSecOps practices mentioned above.


# Enhance security posture of Azure solution architecture

Having DevSecOps practices in place is very important. Above in the article, I mentioned Azure Infrastructure Security. When building Azure solutions it is not enough to focus on code security. We have to make sure that our solution is also secure from the Infrastructure setup side.

Below I present two versions of the sample Azure solution architecture. The first one was created without thinking about the security posture of Azure architecture. The second one presents security-enhanced solution architecture.


<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-05.png?raw=true" alt="Image not found"/>
</p>

As we can see above, there are no security enhancements. How can we improve the security posture of this sample Azure solution architecture?

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-06.png?raw=true" alt="Image not found"/>
</p>

In the above, updated architecture we applied the following enhancements:

1. Resources are now inside Azure Virtual Network and communicate with each other using Microsoft backbone infrastructure, not the public Internet.
2. Direct access to APIs is not allowed, all traffic has to go through Azure Front Door with Web Application Firewall
3. All secrets and credentials are stored in the Azure Key Vault
4. There is Log Analytics with Application Insights used to collect logs
5. Microsoft Sentinel is used to detect potential threats and mitigate them

As you can see, with the above updates we can make our Azure solution more secure.


## Threat modeling for Azure solution

We discussed *Plan and develop* phase above in the article, and one of the elements was Threat Modelling. It allows software architects to identify and mitigate potential security issues early when they are relatively easy and cost-effective to resolve. As a result, it greatly reduces the total cost of development.
I strongly recommend using [Microsoft Threat Modeling Tool](https://docs.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-getting-started).

With threat modeling the approach involves creating a diagram, identifying threats, mitigating them, and validating each mitigation:

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-08.png?raw=true" alt="Image not found"/>
</p>


Here we can see the basic Threat Model I created for the Azure architecture I mentioned above:

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-07.png?raw=true" alt="Image not found"/>
</p>

This is one of the recommendations from the generated report:

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-09.PNG?raw=true" alt="Image not found"/>
</p>


# Summary

In this article, I explained what DevSecOps is and what good practices should be used when building Azure cloud solutions. This topic is huge, and obviously, I did not touch every aspect of DevSecOps on Azure but I hope that this introduction article helped you understand some concepts. Below I present what will be included in the next articles as I planned a series of articles about DevSecOps on Azure.

<p align="center">
<img src="/images/devisland/article89/assets/devsecopsazure-intro-10.PNG?raw=true" alt="Image not found"/>
</p>



*If you like my articles, and find them helpful, please buy me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>