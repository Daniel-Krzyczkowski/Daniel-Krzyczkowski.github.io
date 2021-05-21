---
title: "Auth0 Actions with Azure Function App"
excerpt: "This article presents how to integrate Auth0 Actions with Azure Function App"
header:
  image: /images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps1.png
---

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps1.png?raw=true" alt="Lost in Azure cloud identity - part 8"/>
</p>


# Introduction

Some time ago on the Auth0 blog, there was the announcement that Auth0 Actions is now generally available. You can read more about this announcement [here](https://auth0.com/blog/actions-now-generally-available/).
In this article, I would like to briefly introduce you to the concept of Auth0 Actions and how to integrate them with Azure Function App hosted on Microsoft Azure cloud. I will focus on the specific scenario but I will explain it later in the article.

## Auth0 Actions - what is this?

Actions are secure, tenant-specific, versioned functions written in Node.js that execute at certain points during the Auth0 runtime. To be more specific and make it more clear let me put some examples. What an Action can do is determined by where it is executed within the Auth0 runtime:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps10.PNG?raw=true" alt="Image not found"/>
</p>

As you can see above we can specify actions for specific flows:

### Login 

We can specify an action that will be executed during user authentication. This can be helpful when we need to call an external service during authentication and inject some data in the ID Token. This kind of action is also executed when Refresh Tokens are issued. This can be also helpful when we want to notify the external system once the user is authenticated.

### Machine to machine 

Actions for this kind of flow are executed when an Access Token is issued using the Client Credentials Flow. Again, like mentioned above, we can enrich tokens with additional data.


### Pre User Registration

Actions for this flow are executed before a user is added to a Database or Passwordless Connection. It can be used to verify user data provided during registration and prevent the creation of a user in Auth0.

### Post User Registration

Actions for this flow are executed after a user is added to a Database or Passwordless Connection. This is the flow that we are going to implement in this article. With this flow, we can execute Action to notify the external system that a new user was registered in Auth0.

### Post Change Password

Actions for this flow are executed after a password is changed for a Database Connection user. This kind of flow can be helpful to either send an email to a user to notify them that their password has been changed or to notify external systems to revoke a user’s sessions.


### Send Phone Message

There can be scenarios where a custom provider for sending MFA Phone or SMS messages is required. With this flow and Action added we can achieve it.


**You can read more about details for each flow in the [official Auth0 Actions documentation](https://auth0.com/docs/actions#what-can-you-do-with-actions)**


# Auth0 Action to call external API

In this article, we will learn how to execute Auth0 Action which executes in the *Post User Registration* flow. This Action calls Azure Function App with the newly registered user's identifier once the user is successfully registered in Auth0. Let's start with an explanation of the Azure Function App source code.

## Azure Function App

I created HTTP Trigger Function App in Visual Studio 2019. If you want to learn how to create it, please check [this official, step-by-step documentation](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-your-first-function-visual-studio).

Here is the source code of the Function App I created:

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace Auth0Actions.FuncApp
{
    public static class Function1
    {
        [FunctionName("auth0-actions-user-func-app")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("auth0-actions-user-func-app function triggered");

            string userId = string.Empty;

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            userId = data?.userId;

            log.LogInformation($"Received ID of the newly registered user: {userId}");

            return new OkResult();
        }
    }
}
```

As you can see above, there is *POST* HTTP Trigger Function App. It expects to receive JSON data. In this simple example, I expect to receive *userId* in the body. To extract *userId*, first I deserialize the request body. Then retrieve *userId* field from the *data* dynamic variable. In the last step, I log information about the newly registered user ID with *log.LogInformation($"Received ID of the newly registered user: {userId}")* line.

During the creation of the Azure Function App in the Azure portal, I integrated it with Azure Application Insights. Once *log.LogInformation* line is executed, userId is logged in the Azure Application Insights. This is only for the test purpose. In the real-world scenario, you could store userId in the database or other external system. Please remember that you can also send additional data related to the user profile, not only userId. You can pass information about user email or first name for instance.

Here are the components I created in the Azure portal:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps9.PNG?raw=true" alt="Image not found"/>
</p>

I have an Azure Function App created together with Storage Account, and Azure Application Insights. I have deployed above Function App using Visual Studio. Once the Function App is deployed, it has its own URL address:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps11.PNG?raw=true" alt="Image not found"/>
</p>

We will use this address in the Auth0 Action to call the Function App once the new user is registered.


## Auth0 Action

Now it is time to create new Action. In the Auth0 portal, select *Custom Actions* under *Actions* section. Then click *Create* button:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps12.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps5.PNG?raw=true" alt="Image not found"/>
</p>

To call external API using HTTP request we need to use *axios* - HTTP client for the browser and node.js. Paste below code in the editor:

```js
 const axios = require("axios");

 async function makePostRequestAsync(userId) {

    let payload = { userId: userId};

    let res = await axios.post('https://func-auth0-actions.azurewebsites.net/api/auth0-actions-user-func-app?code=vpGU9pJbfXOmU5rx8TTlTsgpkrX1XzbCzD01pMCdLI1G2gWW5Tdjng==', payload);

    let data = res.data;
    console.log(data);
}

 exports.onExecutePostUserRegistration = async (event) => {

    await makePostRequestAsync(event.user.user_id);
 };
```

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps6.PNG?raw=true" alt="Image not found"/>
</p>


Please note that in the above code I call Azure Function described earlier with *userId* passed in the JSON body. There is also Function URLs used in the *POST* request. Once the code is ready we can save this Action. I called it *register-user-in-external-system*. Then click *Deploy* button to make this Action to be available for the flows:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps13.PNG?raw=true" alt="Image not found"/>
</p>

You should see *Deployed* status:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps18.PNG?raw=true" alt="Image not found"/>
</p>


Once the action is ready we can use it in the *Post User Registration* flow:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps4.PNG?raw=true" alt="Image not found"/>
</p>

Move the Action displayed on the right side to the flow. It should land between *Start* and *Complete* dots.


<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps4.PNG?raw=true" alt="Image not found"/>
</p>

Then click *Apply* button.


## Test the flow

Once we added Action to the *Post User Registration*, we can test the flow. Click *Getting Started* tab, and then *Try it out* link:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps7.PNG?raw=true" alt="Image not found"/>
</p>

Try to register as new user:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps8.PNG?raw=true" alt="Image not found"/>
</p>

Once you succcessfully registered, open *Users* tab:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps15.PNG?raw=true" alt="Image not found"/>
</p>

Find newly created user on the list and click it. You should see the details about the user profile displayed. Please note that there is unique ID of the user displayed:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps3.PNG?raw=true" alt="Image not found"/>
</p>


Now open Azure Application Insights and click *Search* tab:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps16.PNG?raw=true" alt="Image not found"/>
</p>

Then select time range to be last 30 minutes:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps17.PNG?raw=true" alt="Image not found"/>
</p>

Yoy should find the log information with the newly registered user ID:

<p align="center">
<img src="/images/devisland/article73/assets/Auth0-Actions-With-Azure-Function-Apps2.PNG?raw=true" alt="Image not found"/>
</p>

Of course in this scenario, we only logged this information but you could store it in an external database for instance.


# Summary

In this article, I presented how to use Auth0 Actions to call external service once the new user is registered. In this specific example, I used Azure Function App to receive the information about the new User ID. I encourage you to check other Actions and possible flows where they can be used.