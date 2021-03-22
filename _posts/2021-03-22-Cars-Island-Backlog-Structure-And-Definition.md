---
title: "Cars Island – backlog structure and definition - part 12"
excerpt: "This article presents how to use Azure DevOps manage implementation progress for the project"
header:
  image: /images/devisland/article61/assets/CarsIslandBacklog1.jpg
---

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog1.jpg?raw=true" alt="Cars Island – backlog structure and definition - part 12"/>
</p>


# Introduction

In [my first article](https://daniel-krzyczkowski.github.io/Cars-Island-Car-Rental-On-Azure-Cloud/), I introduced you to Cars Island car rental on the Azure cloud. I created this fake project to present how to use different Microsoft Azure cloud services and how to their SDKs. I also presented what will be covered in the next articles. Here is the next article from the series where I would like to discuss how to build the backlog for the Cars Island solution.

Below I present solution architecture:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandAzureInfra2.png?raw=true" alt="Image not found"/>
</p>


*[Cars Island project is available on my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure)*


# Backlog and roadmap

It does not matter what kind of solution we build and what Azure cloud service we are going to use. The central place for managing implementation state and collecting requirements is crucial to make sure that our project is in a good condition when it comes to timelines and agreed functionalities. This is why for the Cars Island project I have created a product backlog using Azure DevOps Boards. If you would like to read more about Azure DevOps functionalities, please check [this](https://azure.microsoft.com/en-us/overview/devops-tutorial/#additional) link.

In the Cars Island solution, there are functionalities like:

1. Displaying list with all available cars
2. Creating reservation for a specific car
3. Sending enquiry with attached file

Features mentioned above should be implemented in both applications - Web API and Web Portal. Of course, we have to also implement integration with the database for instance, or integrate with SendGrid to send reservation confirmation.

# Backlog structure in the Azure DevOps

In the Azure DevOps in the *Boards* section, there is a tab called *Backlog*:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog3.PNG?raw=true" alt="Image not found"/>
</p>

This is the place where we can define our product backlog and collect details about features and implementation state.

This is the backlog I have created for Cars Island:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog4.PNG?raw=true" alt="Image not found"/>
</p>

As you can see there is a specific structure created (I use Agile process for my project in the Azure DevOps):

1. Epics
2. Features
3. User Stories
4. Tasks

If you are interested in different processes available in the Azure DevOps, [here](https://docs.microsoft.com/en-us/azure/devops/boards/work-items/guidance/choose-process?view=azure-devops&tabs=basic-process) is a good source of information.

I used *Epics* to define high-level goals and parts of the system:

1. Cars Island Azure cloud infrastructure
2. Cars Island Backend Services
3. Cars Island Web Portal

Now you can ask if this is one, the best structure. The answer is simple - no. Different solutions can have different structures and functionalities. This is why you should try to create a backlog structure to make it easier to manage, find and control the implementation of specific parts and features of the solution.

Under each *Epic* I have defined *Features*. Example: *Cars Island backend services can send reservation confirmation emails to users*.

Now under this *Feature* I have defined *User Story*: *As a User, I want to receive an email confirmation that my car is reserved*.

Under *User Story* there are *Tasks*:

1. *Create Azure Function template with Azure Service Bus Queue item trigger*
2. *Integrate Azure Function with Azure SendGrid C# SDK to send emails*

Each backlog item has its state and person that is responsible for implementation (in the Cars Island the only person was me):

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog5.PNG?raw=true" alt="Image not found"/>
</p>


# Iterations

One of the great functionalities provided by Azure DevOps is related to iterations. We can decide which backlog work items will be implemented first and when the rest of them will be implemented. *Iterations* can be defined under project configuration in the Azure DevOps:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog6.PNG?raw=true" alt="Image not found"/>
</p>

As you can see we can also set dates for these iterations. Once iterations are set, we can decide which *User Stories* we are going to implement in each iteration:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog7.PNG?raw=true" alt="Image not found"/>
</p>

Once you set iterations, you can open *Sprints* tab and see all the *User Stories* for the specific iteration:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog8.PNG?raw=true" alt="Image not found"/>
</p>


<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog9.PNG?raw=true" alt="Image not found"/>
</p>


# Queries

One of the very helpful features are *Queries*. With *Queries* we can filter work items in our backlog:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog10.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog11.PNG?raw=true" alt="Image not found"/>
</p>

This can be helpful when we want to quickly filter out some specific work items from our backlog without changing its structure.


# Dashboards

Dashboards in the Azure DevOps help you display some different details about the project condition in one place. You can add multiple widgets. In the Cars Island project I defined widgets to display information about releases:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog12.PNG?raw=true" alt="Image not found"/>
</p>

Of course, you can add more widgets if you want and customize the dashboard. You can also create multiple dashboards.


# Wiki

Wikis in the Azure DevOps can be helpful when it comes to collecting information about implementation details, architecture and decissions. For the Cars Island I created Wiki where I store information about the architecture, and informaton about deployment scripts usage:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog13.PNG?raw=true" alt="Image not found"/>
</p>

Cool feature is an option to keep Wiki in the GIT repository. Using it in this way we can review the proposed changes before the will apprear officially in the Wiki:

<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog15.PNG?raw=true" alt="Image not found"/>
</p>


<p align="center">
<img src="/images/devisland/article61/assets/CarsIslandBacklog14.PNG?raw=true" alt="Image not found"/>
</p>



# Summary

In this article, I described how to use Azure DevOps to create product backlog and organize work. Functionalities like dashboards and wiki can be really helpful to collect information about the project. Source code of the Cars Island solution is available on [my GitHub](https://github.com/Daniel-Krzyczkowski/Cars-Island-On-Azure) so you can see all implementation details. If you want to learn more about Azure DevOps, I recommend checking [official documentation](https://docs.microsoft.com/en-us/azure/devops/?view=azure-devops). I also encourage you to read my other article called [Release notes with Azure Functions and Azure DevOps](https://daniel-krzyczkowski.github.io/Release-Notes-With-Azure-Functions-And-Azure-DevOps/) where I showed how to publish release notes to Wiki.
