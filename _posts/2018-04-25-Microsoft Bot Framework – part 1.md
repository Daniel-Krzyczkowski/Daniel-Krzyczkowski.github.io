﻿---
title: "Microsoft Bot Framework – part 1"
excerpt: "In this article I would like to describe what exactly Microsoft Bot Framework offers and what are its key concepts and present how to create simple Bot in Azure portal."
header:
  image: /images/devisland/article2/assets/botframeworkcover.png
---

<p align="center">
<img src="/images/devisland/article2/assets/botframeworkcover.png?raw=true" alt="Microsoft Bot Framework – part 1"/>
</p>

<h3><strong>Short introduction</strong></h3>
Microsoft Bot Framework enables developers to create intelligent applications to communicate with users. In this article I would like to describe what exactly Microsoft Bot Framework offers and what are its key concepts and present how to create simple Bot in Azure portal.

Bot is an application which enables users to communicate with it with text, cards or speech.

Basically there are two types of bots:

a. Simple bot with pattern matching – user sends message, bot analysis request and check whether there is a possible response

b. More sophisticated bot which is using artificial intelligence including advanced connections and state tracking

Sample conversation can look like:

<img src="/images/devisland/article2/assets/conversationsample.png?w=300" alt="" width="300" height="245" />
<h3><strong>Dictionary</strong><strong> </strong></h3>
<strong>Microsoft Azure</strong>

Microsoft Cloud Platform which enables set of cloud services that developers and IT professionals use to build, deploy, and manage applications through our global network of datacenters. One of available services is Bot Service described below.

<strong>Microsoft Azure Bot Service</strong>

Microsoft Azure Bot Service provides what you need to build, connect, test, deploy, monitor, and manage bots. Bot Service provides the core components for creating bots, including the Bot Builder SDK (described below) for developing bots and the Bot Framework for connecting bots to channels.

Bot Service provides an integrated environment purpose-built for bot development. You can write a bot, connect, test, deploy, and manage it from your web browser with no separate editor or source control required. Simple bots may not need to write code at all.

&nbsp;

Bot Service provides two possible hosting plans:

<img src="/images/devisland/article2/assets/botwebapp.png?w=96&amp;h=96&amp;crop=1" alt="" width="96" height="96" />

With the App Service plan, a bot is a standard Azure web app you can set to allocate a predefined capacity with predictable costs and scaling.

<img src="/images/devisland/article2/assets/botfunctionapp.png?w=96&amp;h=96&amp;crop=1" alt="" width="96" height="96" />

With a Consumption plan, a bot is a serverless bot that runs on Azure Functions and uses the pay-per-run Azure Functions pricing.

<img src="/images/devisland/article2/assets/bot01additional.png?w=775" alt="" width="775" height="117" />

<strong>Bot Builder</strong>

The Bot Builder provides an SDK, libraries, samples, and tools to help you build and debug bots. When you build a bot with Bot Service (described above), your bot is backed by the Bot Builder SDK. You can also use the Bot Builder SDK to create a bot from scratch using C# or Node.js

<strong>Bot REST API</strong>

Developers can create a bot with any programming language by using the Bot Framework REST API.

Three REST APIs in the Bot Framework are available:

a. <strong>The Bot Connector REST API </strong>(<a href="https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference">documentation</a>) enables your bot to send and receive messages to channels configured in the Bot Framework Portal.

b. <strong>The Bot State REST API</strong> enables your bot to store and retrieve state associated with the conversations that are conducted through the Bot Connector REST API.
IMPORTANT note from official documentation:

The Bot Framework State Service API is not recommended for production environments, and may be deprecated in a future release. It is recommended that you update your bot code to use the in-memory storage for testing purposes or use one of the Azure Extensions for production bots.

c. <strong>The Direct Line REST API </strong>(<a href="https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts">documentation</a>) enables you to connect your own application, such as a client application, web chat control, or mobile app, directly to a single bot.

&nbsp;

<strong>Bot Framework Emulator</strong>

The <a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator">Bot Framework Emulator</a> is a desktop application that allows you to test and debug you bots, either locally or remotely. Now when you know what is Bot, Bot Service and Bot Builder it is time to discuss two possible options to create your Bot.

There are two possible ways to create the Bot:

a. Using ready-to-go template available in Azure portal – with this approach you need to setup new Bot Service in Azure portal and at the end you have access to Bot source code for further modifications (if needed)
b. Writing your Bot from scratch in Visual Studio using Bot Builder SDK and then publish it to Azure
<strong> </strong>

<strong>Create Bot with Bot Service available through Microsoft Azure portal</strong>

In this section I would like to present how to create Bot through Microsoft Azure portal using Bot Service and App Service Plan.

a. Sign in to Azure <a href="http://portal.azure.com">portal</a> with your Microsoft Account (Azure subscription is required here)
b. Click “new” and select “AI + Cognitive Services”
c. Select “Web App Bot”

<img src="/images/devisland/article2/assets/azurebotservice1.png?w=300" alt="" width="300" height="242" />

d. Type the unique name of your bot, select subscription, resource group, location and pricing tier. Type the name of web application (it can be the same as Bot name from first field).

<img src="/images/devisland/article2/assets/azurebotservice2.png?w=213" alt="" width="213" height="300" />

e. In the next section you can select bot template. Template is start point for your bot. Basing on your choice selected source code is selected for bot which you can develop further with Bot Builder SDK. Select “basic” template.

<img src="/images/devisland/article2/assets/azurebotservice3.png?w=300" alt="" width="300" height="181" />

f. Last thing is to setup App Service plan, create new Azure Storage (to store Bot data) and select location for Application Insights (to monitor performance of the Bot). At the end click “Create” button.

<img src="/images/devisland/article2/assets/azurebotservice4.png?w=200" alt="" width="200" height="300" />

<img src="/images/devisland/article2/assets/azurebotservice5.png?w=300" alt="" width="300" height="92" />

&nbsp;

g. Bot is ready for tests. Select “Test in Web Chat” section.

<img src="/images/devisland/article2/assets/azurebotservice6.png?w=300" alt="" width="300" height="146" />

<img src="/images/devisland/article2/assets/azurebotservice7.png?w=218" alt="" width="218" height="300" />

<img src="/images/devisland/article2/assets/azurebotservice8.png?w=300" alt="" width="300" height="197" />

Of course you have full access to Bot source code. Select “Build” tab.

a. You can open code in online editor:

<img src="/images/devisland/article2/assets/azurebotservice10.png?w=300" alt="" width="300" height="163" />

b. Another option (more powerful) is to download Bot source code and edit it in Visual Studio:

<img src="/images/devisland/article2/assets/azurebotservice11.png" alt="" width="217" height="197" />

You can also configure additional channels to communicate with your Bot like Microsoft Teams, Facebook Messenger or Skype.

<img src="/images/devisland/article2/assets/azurebotservice12.png?w=300" alt="" width="300" height="193" />

Another nice feature is configuration where you can set Bot icon, title and description.

<img src="/images/devisland/article2/assets/azurebotservice13.png?w=300" alt="" width="300" height="243" />
<h3><strong>Wrapping up</strong></h3>
Microsoft Bot Framework enables creating smart applications which can communicate with users with natural conversation flow. Developers are able to write specific functionality of bots using Bot Builder SDK available for NET C# and Node.js and host their bots on Microsoft Azure platform. If you would like to read more please refer to official <a href="https://docs.microsoft.com/en-us/azure/bot-service/">documentation.</a>

In the next part I will show how to extend bot functionality and discuss bot’s source code opened in Visual Studio.
