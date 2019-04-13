---
title: "Microsoft Bot Framework – part 3"
---

<p align="center">
<img src="/images/devisland/article4/assets/botframeworkcover.png?raw=true" alt="Microsoft Bot Framework – part 3"/>
</p>


<h3><strong>Short introduction</strong></h3>
Microsoft Bot Framework enables developers to create intelligent applications to communicate with users. In the first part I described some key concepts connected with Bot Framework, in the second part I went through Bot source code and in this article I would like to discuss how to use Bot Emulator and how to connect Bot to different channels like Skype and how to communicate with it.
<h3><strong>Bot Emulator</strong></h3>
<p style="text-align:center;"><img class="alignnone wp-image-193" src="/images/devisland/article4/assets/botf25.png?w=300" alt="" width="419" height="315" /></p>
It is possible to test the Bot with Bot Emulator. You can download it from GitHub <a href="https://github.com/Microsoft/BotFramework-Emulator/releases" target="_blank" rel="noopener">here.</a>

Important note here - if you are using Windows and you are running the Bot Framework Emulator behind a firewall or other network boundary and want to connect to a bot that is hosted remotely, you must install and configure ngrok tunneling software. You can read how to install and configure ngrok <a href="https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator" target="_blank" rel="noopener">here.</a>

Now if you launch your Bot from Visual Studio on localhost you can connect to it and test it. Before conversation you have to authenticate with Microsoft App Id and Microsoft App Password:
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf26.png?w=300" alt="" width="383" height="190" /></p>
Now you can talk to your Bot. On the right side there is log displayed:

<img src="/images/devisland/article4/assets/botf28.png?w=300" alt="" width="387" height="237" />

You can also publish your Bot on Azure and access it from the emulator:

<img src="/images/devisland/article4/assets/botf29.png?w=300" alt="" width="377" height="235" />

&nbsp;
<h3><strong>Integrate Bot with different channels</strong></h3>
Microsoft Azure Bot Service enables possibility to integrate Bot with channels like Skype, Microsoft Teams or Slack.
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf12.png?w=300" alt="" width="471" height="272" /></p>
Below I would like to present how to integrate Bot to Skype channel.

In Azure Portal after selecting Bot there is "Channels" tab available:
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf13.png?w=118" alt="" width="150" height="382" /></p>
Once clicked card with possible channels to integrate is displayed. In this case I selected "Skype" channel:
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf14.png?w=300" alt="" width="300" height="174" /></p>
There is a possibility to embed Skype control on your own website if you prefer such form of communication:
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf15.png?w=300" alt="" width="446" height="253" /></p>
Send tab called "Messaging" presents options how you can communicate with the Bot. There are two possible options to select:
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf16.png?w=300" alt="" width="428" height="227" /></p>
a. Messaging - standard way to send and receive messages
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf17.png?w=300" alt="" width="359" height="103" /></p>
b. Media cards - to display rich media content like videos
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf18.png?w=300" alt="" width="352" height="108" /></p>
It is possible to call to the Bot and talk with it. This can be enabled in "Calling" tab:
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf19.png?w=300" alt="" width="354" height="316" /></p>
In the "Groups" tab you it is possible to determine if your bot can be added to a group and how it behaves in a group for messaging and is also used to enable Group Video Calls for Calling bots:

<img src="/images/devisland/article4/assets/botf20.png?w=300" alt="" width="385" height="185" />

Last tab called "Publish" requires providing details connected with Bot before publication like name and privacy description:
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf21.png?w=300" alt="" width="362" height="260" /></p>
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf22.png?w=220" alt="" width="264" height="360" /></p>
<img src="/images/devisland/article4/assets/botf23.png?w=300" alt="" width="319" height="230" />

Bots in Preview are limited to 100 contacts. If you need more than 100 contacts, your have to submit your bot for review.

Once you click "Save" button you have to accept Terms of Service.

Skype should now be displayed as one of possible channels to communicate with Bot:
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf24.png?w=300" alt="" width="367" height="225" /></p>
You can click on Skype channel, sign in with your Microsoft account and add the Bot to your contacts:
<p style="text-align:center;"><img src="/images/devisland/article4/assets/botf30.png?w=300" alt="" width="360" height="293" /></p>
&nbsp;
<h3><strong>Wrapping up</strong></h3>
That was the last part of Microsoft Bot Framework series. I hope that these three articles help you to start your journey with Microsoft Azure Bot Service.

If you would like to read more please refer to official <a href="https://docs.microsoft.com/en-us/azure/bot-service/">documentation.</a>