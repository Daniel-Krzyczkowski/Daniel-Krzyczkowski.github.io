---
title: "Real-time data with Microsoft Azure SignalR Service"
---

<p align="center">
<img src="/images/devisland/article8/assets/cortanaskills1.png?raw=true" alt="Real-time data with Microsoft Azure SignalR Service"/>
</p>

<h3><strong>Short introduction</strong></h3>
Imagine digital assistant which supports you with different tasks like booking hotel or ordering taxi. Imagine that you have access to it through the phone, personal computer or tablet. Yes it is possible with personal digital assistant from Microsoft called Cortana. You can communicate with it by text or voice. Sounds unreal? Lets dive into Cortana Skills Kit.
<h3><strong>Cortana configuration</strong></h3>
Cortana is available in Windows 10 system (but not only - iOS and Android platforms are supported too) and it is so simple to enable it.

Click "Windows" button, type "Cortana" and select "Cortana &amp; Search settings":

<img class=" wp-image-669 aligncenter" src="/images/devisland/article8/assets/cortanaskills2.png?w=189" alt="" width="269" height="427" />

Enable "Let Cortana respond to Hey Cortana":
<p style="text-align:center;"><img class="wp-image-671 aligncenter" src="/images/devisland/article8/assets/cortanaskills3.png?w=300" alt="" width="274" height="245" /></p>
That's it! Now say to your computer "Hey Cortana!". Remember to enable sound. You should see that Cortana virtual assistant appeared:

<img class=" wp-image-673 aligncenter" src="/images/devisland/article8/assets/cortanaskills4.png?w=300" alt="" width="462" height="261" />

Currently Cortana is available in english only. Once Cortana is configured we can start exploring Cortana Skills Kit.
<h3><strong>Cortana's Ecosystem</strong></h3>
Before I describe Cortana Skills Kit and show it's capabilities I would likt to present this nice slide presented during the MS Build 2018 session related with Cortana's Ecosystem:

<img class=" wp-image-678 aligncenter" src="/images/devisland/article8/assets/cortanaskills5.png?w=300" alt="" width="582" height="328" />

As you can see Cortana can be accessed from many devices and platforms. Pay attention to Cortana Skills Kit part.
<h3><strong>Cortana Skills Kit</strong></h3>
Before I describe the solution it is worth to start from clarification - what is a sklill? As mentioned during MS Build 2018 "skill is a unit of conversational intelligence that enables Cortana to help users using their services".

<img class=" wp-image-686 aligncenter" src="/images/devisland/article8/assets/cortanaskills6.png?w=300" alt="" width="588" height="304" />

Now Cortana Skills Kit is a suite of tools that help developers build skills that connect users to their custom services and solutions.

There are two components from which Skills Kit consists:
<p style="text-align:left;"><strong>- </strong><strong>Microsoft Bot Framework</strong> - framework which enables developers to create intelligent application which communicate with users through different channels like Skype or Cortana. You can read more in my previous article <a href="https://devislandblog.wordpress.com/2018/04/24/microsoft-bot-framework-part-1/" target="_blank" rel="noopener">here.</a> You can for instance create bot which can make flight booking.</p>
<strong>- Cortana Skills Dashboard</strong> - dashboard where you can manage Cortana skills. This is the place where you can deploy skill to yourself (for testing purpose), a group of users, or the world. This is also the place where you can publish your bot created with Bot Framework so it can be available through Cortana for all users. You can access dashboard <a href="https://my.knowledge.store/" target="_blank" rel="noopener">here.</a>

To clarify - creating bot is <strong>not required.</strong> You can directly create skill in Cortana Skills Dashboard and decide how conversation flow will look like. Cortana Skills Kit is currently in public preview at the moment of writing this article so some elements can change in the future.

To make it more understandable I presented below how to create skill as bot created with Microsoft Bot Framework and without it using Semantic Composition Language (SCL) in Cortana Skills Dashboard (also called Knowledge.Store).
<h3><strong>Create skill as Bot Framework chat bot
</strong></h3>
Bot Framework enables creating skill logic. In this section I will create simple bot with Microsoft Bot Framework which will be connected with Language Understanding Intelligence Service (LUIS).

This will be medical bot which ask questions related with user's health.

Users can communicate with it through Cortana (by text or voice). In this case skill is a bot hosted on Microsoft Azure cloud. Below diagram presents steps we will do in this section:

<img class="wp-image-713 aligncenter" src="/images/devisland/article8/assets/cortanaskills91.png?w=300" alt="" width="545" height="78" />

1. Login to <a href="https://portal.azure.com" target="_blank" rel="noopener">Azure portal</a> and search for "Bot" in Marketplace:

<img class=" wp-image-725 aligncenter" src="/images/devisland/article8/assets/cortanaskills12.png?w=300" alt="" width="467" height="196" />

2. Fill all required configuration:

<img class=" wp-image-727 aligncenter" src="/images/devisland/article8/assets/cortanaskills13.png?w=73" alt="" width="189" height="774" />

Please especially pay attention to bot template section. I selected "Language Understanding" template because I will integrate my bot with Language Understanding Intelligence Service (LUIS):

<img class=" wp-image-730 aligncenter" src="/images/devisland/article8/assets/cortanaskills14.png?w=224" alt="" width="344" height="461" />

Second important note - LUIS app location should be the same as location of the bot (so in my case West Europe).

Click "Create" until resources are ready.

<img class=" wp-image-738 aligncenter" src="/images/devisland/article8/assets/cortanaskills151.png?w=300" alt="" width="636" height="244" />

&nbsp;

3. Now it is time to configure LUIS and add some intents. LUIS is one of the cognitive services from Microsoft Azure that enables applications to understand what the user is saying in natural language.

Sign in to <a href="https://eu.luis.ai" target="_blank" rel="noopener">LUIS portal</a> and click "My apps". Before you do this please check URL address for you region. In my case I selected West Europe location so I should sign in under: <strong>eu.luis.ai</strong>

You can check URL for your region <a href="https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions" target="_blank" rel="noopener">here</a>.

<img class=" wp-image-742 aligncenter" src="/images/devisland/article8/assets/cortanaskills16.png?w=300" alt="" width="655" height="175" />

You should see "Intents" section with the list:

<img class="wp-image-751 aligncenter" src="/images/devisland/article8/assets/cortanaskills17.png?w=300" alt="" width="666" height="244" />

Before I go through them it is worth to explain some key concepts related with LUIS.

<strong>Intent</strong>

An intent represents actions the user wants to perform. The intent is a purpose or goal expressed in a user's input, such as booking a flight or paying a bill. You have to define and name intents that correspond to these actions. A support app may define an intent named "GetComputer."

<strong>Utterance</strong>

An utterance is text input from the user that application needs to understand. It may be a sentence, like "Where can I get my computer?", or a fragment of a sentence, like "Computer" or "Computer order".

<strong>Entiy</strong>

An entity represents detailed information that is relevant in the utterance. For instance, in the utterance "Order new computer on 25 of January", "25 of January" is a date here. Entities which are mentioned in the user’s utterance can help LUIS to choose the specific action to take to answer a user's request.

Five types of entities are available:

<strong>Simple</strong>

A simple entity is a generic entity that describes a single concept like computer or order.

<strong>Hierarchical</strong>

A hierarchical entity is a special type of a simple entity; defining a category and its members in the form of parent-child relationship. There can be Location entity with FromLocation and ToLocation child entities.

<strong>Composite</strong>

A composite entity is made up of other entities, such as prebuilt entities, list entities, and simple. The separate entities form a whole entity. PlaneTicketOrder is an example of such entity because it can have Number and ToLocation child entities.

<strong>List</strong>

List entities represent a fixed, closed set (white list) of related words in your system. City list is a good example because it allows variations of city names including city of airport (Sea-tac), airport code (SEA), postal zip code (98101), and phone area code (206).

<strong>Regex</strong>

A regular expression (regex) entity ignores case and ignores cultural variant. For instance regex entity kb[0-9]{6,} matches kb123456.

As I mentioned on the beginning of this section this will be medical bot which can ask questions related with user's health. I will add some key intents by clicking "Create new intent". Here are my intents:

- SoarThroat

- HeadAke

- TwistedAnke

<img class=" wp-image-757 aligncenter" src="/images/devisland/article8/assets/cortanaskills18.png?w=300" alt="" width="561" height="342" />

Rest of the intents can stay as they are. For each of my three intents I will add five utterances which user can send to bot.

For HeadAke intent I applied below utterances:

<img class=" wp-image-760 aligncenter" src="/images/devisland/article8/assets/cortanaskills19.png?w=300" alt="" width="552" height="353" />

For SoarThroat intent I applied below utterances:

<img class=" wp-image-763 aligncenter" src="/images/devisland/article8/assets/cortanaskills20.png?w=300" alt="" width="547" height="449" />

For TwistedAnkle intent I applied below utterances:

<img class=" wp-image-766 aligncenter" src="/images/devisland/article8/assets/cortanaskills21.png?w=300" alt="" width="570" height="304" />

Now it is time to train LUIS so it can corrrectly discover intents from utterances sent by users. Click "Train" button:

<img class=" wp-image-768 aligncenter" src="/images/devisland/article8/assets/cortanaskills22.png?w=300" alt="" width="589" height="92" />

You should see confirmation after few seconds:

<img class=" wp-image-771 aligncenter" src="/images/devisland/article8/assets/cortanaskills23.png?w=300" alt="" width="577" height="96" />

Now click "Test" and type utterance: "I feel pain in my head". You should see that LUIS analyzed utterance and matched it with "HeadAke" intent:

<img class=" wp-image-773 aligncenter" src="/images/devisland/article8/assets/cortanaskills24.png?w=300" alt="" width="569" height="248" />

Now click "Publish" section and then click "Publish" button:

<img class=" wp-image-775 aligncenter" src="/images/devisland/article8/assets/cortanaskills25.png?w=300" alt="" width="575" height="115" />

LUIS application is successfully published. Now we can connect to it from the bot. Before we move forward please copy AppID and Key from "Resources and Keys" section on the same page. Remember to switch to your region (in my case West Europe):

<img class=" wp-image-777 aligncenter" src="/images/devisland/article8/assets/cortanaskills26.png?w=300" alt="" width="570" height="108" />

4. Verify app settings of the bot. It is worth to check that correct LUIS Key and AppID is added in application settings. To verify it click Web App Bot in Azure portal:

<img class=" wp-image-779 aligncenter" src="/images/devisland/article8/assets/cortanaskills27.png?w=300" alt="" width="589" height="106" />
<p style="text-align:center;"><img class="alignnone wp-image-782" src="/images/devisland/article8/assets/cortanaskills28.png?w=300" alt="" width="591" height="319" /></p>
5. Download and update bot source code

Go to "Build" tab and click "Download zip file":

<img class=" wp-image-785 aligncenter" src="/images/devisland/article8/assets/cortanaskills29.png?w=300" alt="" width="575" height="291" />

Extract zip file and open project in Visual Studio.

<img class=" wp-image-787 aligncenter" src="/images/devisland/article8/assets/cortanaskills30.png?w=300" alt="" width="431" height="374" />

First of all add below settings inside <em>appSettings </em>tag in WebConfig file:

[code language="csharp"]
  <appSettings>
   <add key="MicrosoftAppId" value="" />
   <add key="MicrosoftAppPassword" value="" />
   <add key="LuisAppId" value="" />
   <add key="LuisAPIKey" value="" />
   <add key="LuisAPIHostName" value="" />
   <add key="AzureWebJobsStorage" value="" />
  </appSettings>
[/code]

You can get all these values from Azure portal, from "Application settings" tab mentioned earlier:

<img class="wp-image-791 aligncenter" src="/images/devisland/article8/assets/cortanaskills31.png?w=300" alt="" width="697" height="253" />

Now open "LuisDialog.cs" file. You can see that there is pre-definied code for default intents from LUIS portal:

[code language="csharp"]
// Go to https://luis.ai and create a new intent, then train/publish your luis app.
// Finally replace "Greeting" with the name of your newly created intent in the following handler
        [LuisIntent("Greeting")]
        public async Task GreetingIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Cancel")]
        public async Task CancelIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Help")]
        public async Task HelpIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }
[/code]

Now lets add three additional intents I defined in the LUIS portal: TwistedAnkle, HeadAke and SoarThroat. I also refactored the code so bot now can individually answer each utterance send by users connected with these three intents:

[code language="csharp"]
 [Serializable]
    public class BasicLuisDialog : LuisDialog<object>
    {
        public BasicLuisDialog() : base(new LuisService(new LuisModelAttribute(
            ConfigurationManager.AppSettings["LuisAppId"], 
            ConfigurationManager.AppSettings["LuisAPIKey"], 
            domain: ConfigurationManager.AppSettings["LuisAPIHostName"])))
        {
        }

        [LuisIntent("Greeting")]
        public async Task GreetingIntent(IDialogContext context, LuisResult result)
        {
            var reply = context.MakeMessage();
            reply.Speak = "Hello! How can I help you?";
            reply.Text = "Hello! How can I help you?";

            await context.PostAsync(reply);
            context.Wait(MessageReceived);
        }

        [LuisIntent("SoarThroat")]
        public async Task SoarThroatIntent(IDialogContext context, LuisResult result)
        {
            var reply = context.MakeMessage();
            reply.Speak = "If it is soar throat I think that you should drink hot tea.";
            reply.Text = "If it is soar throat I think that you should drink hot tea.";
            reply.Attachments.Add(new Attachment()
            {
                ContentUrl = "https://assets.tetrapak.com/static/publishingimages/find-by-food/rollup/juice-drinks-tea.jpg",
                ContentType = "image/jpg",
                Name = "Tea.png"
            });

            await context.PostAsync(reply);
            context.Wait(MessageReceived);
        }

        [LuisIntent("HeadAke")]
        public async Task HeadAkeIntent(IDialogContext context, LuisResult result)
        {
            var reply = context.MakeMessage();
            reply.Speak = "If it is head ake I think that you should get aspirine.";
            reply.Text = "If it is head ake I think that you should get aspirine.";
            reply.Attachments.Add(new Attachment()
            {
                ContentUrl = "https://atlas-content-cdn.pixelsquid.com/assets_v2/11/1158052823821195061/jpeg-600/G02.jpg",
                ContentType = "image/jpg",
                Name = "Aspirin.png"
            });

            await context.PostAsync(reply);
            context.Wait(MessageReceived);
        }

        [LuisIntent("TwistedAnkle")]
        public async Task TwistedAnkleIntent(IDialogContext context, LuisResult result)
        {
            var reply = context.MakeMessage();
            reply.Speak = "If it is twisted ankle I think should call your doctor and go to the hospital.";
            reply.Text = "If it is twisted ankle I think should call your doctor and go to the hospital.";
            reply.Attachments.Add(new Attachment()
            {
                ContentUrl = "https://injuryhealthblog.com/wp-content/uploads/2017/11/shutterstock_227120620-min.jpg",
                ContentType = "image/jpg",
                Name = "Ankle.png"
            });

            await context.PostAsync(reply);
            context.Wait(MessageReceived);
        }
    }
[/code]

Please note that each reply has "Text", "Speak" and "Attachments" properties. This is because we want Cortana to say response to the user, write answer on Cortana's canvas and also display attachment - in this case image.

To build the bot I am using Bot Builder SDK available for .NET C#. You can read more <a href="https://docs.microsoft.com/en-us/azure/bot-service/dotnet/bot-builder-dotnet-overview?view=azure-bot-service-3.0" target="_blank" rel="noopener">here</a> about it.

I also published whole source code of my simple medical bot on my <a href="https://github.com/Daniel-Krzyczkowski/MicrosoftAzure/tree/master/DevIslandMedicalBot" target="_blank" rel="noopener">GitHub.</a> There is also great <a href="https://docs.microsoft.com/en-us/azure/bot-service/dotnet/bot-builder-dotnet-concepts?view=azure-bot-service-3.0" target="_blank" rel="noopener">documentation related with Bot Builder.</a>

6. Now it is time to deploy bot source code on Azure. Right click on solution in Visual Studio and click "Publish":

<img class="wp-image-805 aligncenter" src="/images/devisland/article8/assets/cortanaskills33.png?w=198" alt="" width="371" height="561" />

<img class=" wp-image-811 aligncenter" src="/images/devisland/article8/assets/cortanaskills351.png?w=300" alt="" width="501" height="376" />

<img class="alignnone wp-image-807 aligncenter" src="/images/devisland/article8/assets/cortanaskills34.png?w=300" alt="" width="500" height="292" />

Once publishing is completed browser should open:

<img class=" wp-image-814 aligncenter" src="/images/devisland/article8/assets/cortanaskills361.png?w=300" alt="" width="486" height="266" />

6. Get back to the Azure portal and enable "Speech priming" - to improve speech recognition accuracy. You have to paste your LUIS app id here and click "Save":

<img class=" wp-image-816 aligncenter" src="/images/devisland/article8/assets/cortanaskills37.png?w=300" alt="" width="509" height="229" />

7. Enable Cortana channel so users can communicate with your bot through Cortana smart assistant:

<img class=" wp-image-819 aligncenter" src="/images/devisland/article8/assets/cortanaskills38.png?w=300" alt="" width="496" height="379" />

Fill required details as "Display Name", "Description" or "Primary category". You ca put it like in the picture below. Then click "Save" button:

<img class=" wp-image-821 aligncenter" src="/images/devisland/article8/assets/cortanaskills39.png?w=156" alt="" width="381" height="730" />

After few seconds you should see that Cortana channel was configured for the bot. It is time to test it!

<img class=" wp-image-825 aligncenter" src="/images/devisland/article8/assets/cortanaskills40.png?w=300" alt="" width="608" height="136" />

<strong>Talk to your bot through Cortana</strong>

IMPORTANT! Make sure you are signed in to Cortana with the same Microsoft account that you used to register your bot.

If you enabled Cortana (I presented how to do it on the beginning of this article) you can try to say: "Het Cortana, ask &lt;Your Bot Name&gt; about twisted ankle". You should see that Cortana asks for permission. Click "YES":

<img class=" wp-image-827 aligncenter" src="/images/devisland/article8/assets/cortanaskills41.png?w=300" alt="" width="504" height="428" />

Cortana will contect to the bot hosted on Azure:

<img class=" wp-image-830 aligncenter" src="/images/devisland/article8/assets/cortanaskills42.png?w=171" alt="" width="322" height="565" />

You can say "I feel pain in my leg" and see how Cortana reacts:

<img class=" wp-image-833 aligncenter" src="/images/devisland/article8/assets/cortanaskills43.png?w=300" alt="" width="724" height="483" />

I published short video to present whole concept.
<h3><strong>Create skill with Semantic Composition Language (SCL)
</strong></h3>
<img class="wp-image-717 aligncenter" src="/images/devisland/article8/assets/cortanaskills10.png?w=300" alt="" width="514" height="108" />

In this section I would like to present how to create Cortana skill using Semantic Composition Language (SCL).

Before I move forward it is worth to explain some key concepts related with SCL and Cortana skills dictionary.

<strong>Botlets</strong>

<a href="https://help.knowledge.store/system_concepts/botlets/index.html#botlets" target="_blank" rel="noopener">Botlets</a> are the building blocks of a conversational flow. They contain part of the flow logic. They are reusable and can be chained together to create a more complex user experience. Please refer to documentation to see how to create botlet.

<strong>Semantic Composition Language (SCL)</strong>

<a href="https://help.knowledge.store/system_concepts/sculanguage_v3/index.html#about-the-scl" target="_blank" rel="noopener">Semantic Composition Language</a>  (SCL - pronounced skill) is a procedural domain specific language for botlet development where the flow reads in a linear fashion and enables developers to stitch together components from different sources.

<strong>Microsoft Ontology (MSO)</strong>

<a href="https://help.knowledge.store/system_concepts/mso/index.html#microsoft-ontology-mso" target="_blank" rel="noopener">MSO</a>  (Microsoft Ontology) is a set of entity definitions which describe many real-world objects, such as people, restaurant reservations, movies, or stock prices. Each MSO entity includes properties to define the features of the item it classifies. For example, mso.person includes fields for the given_name, family_name, gender, and occupation.

For more please refer to great <a href="https://help.knowledge.store/getting_started/index.html" target="_blank" rel="noopener">documentation</a> related with skills.

1. Login to <a href="https://my.knowledge.store" target="_blank" rel="noopener">Knowledge.Store</a>

2. Create new botlet:

<img class=" wp-image-853 aligncenter" src="/images/devisland/article8/assets/cortanaskills45.png?w=300" alt="" width="515" height="347" />

3. Select "Botlet":

<img class=" wp-image-854 aligncenter" src="/images/devisland/article8/assets/cortanaskills46.png?w=300" alt="" width="514" height="417" />

4. Once you fill all required fields you can start using SCL language to define the skill. In my case I created similar conversation flow as with my medical bot:

<img class=" wp-image-856 aligncenter" src="/images/devisland/article8/assets/cortanaskills47.png?w=300" alt="" width="614" height="313" />

This is my conversation flow defined. Cortana will welcome user and thank ask what happened. Basing on the user input Crotana suggests what to do.

[code language="csharp"]
SAY "Hello! How can I help you?"
GET_INPUT


SAY "So you said: ${USER_INPUT} - OK I will try to help."


CHOICES title = "Please tell me if you feel pain in one of your body parts below:"
  id = "choice_1", text = "LEG"
  id = "choice_2", text = "SOAR"
  id = "choice_3", text = "HEAD"

choice_1:
SAY "If it is twisted ankle I think should call your doctor and go to the hospital."
GO display_image_leg

choice_2:
SAY "If it is soar throat I think that you should drink hot tea."
GO display_image_soar

choice_3:
SAY "If it is head ake I think that you should get aspirine."
GO display_image_head

display_image_soar:
IMAGE "https://atlas-content-cdn.pixelsquid.com/assets_v2/11/1158052823821195061/jpeg-600/G02.jpg"

display_image_head:
IMAGE "https://images.emedicinehealth.com/images/4453/4453-4463-9005-25019tn.jpg"

display_image_leg:
IMAGE "https://injuryhealthblog.com/wp-content/uploads/2017/11/shutterstock_227120620-min.jpg"
[/code]

5. Publish skill so you can invoke it from Cortana. In "Publish" tab type:

<strong>Display name</strong> - this is how your botlet will be named in the Knowledge Store.

<strong>Invocation Name</strong> - this is how you will invoke the skill. In my case I will invoke it saying to Cortana: "Hey Cortana, ask Digital Doctor about twisted ankle".

<img class=" wp-image-859 aligncenter" src="/images/devisland/article8/assets/cortanaskills48.png?w=300" alt="" width="668" height="564" />

6. Invoke skill from Cortana:

<img class=" wp-image-862 aligncenter" src="/images/devisland/article8/assets/cortanaskills49.png?w=300" alt="" width="664" height="442" />

Do not hesitate to use my source code for botlet. That's it!
<h3><strong>Deploy Cortana Skills
</strong></h3>
You can publish created skills (including bot) on three levels:

<img class="wp-image-720 aligncenter" src="/images/devisland/article8/assets/cortanaskills11.png?w=300" alt="" width="689" height="218" />

Skills can be published through <a href="https://my.knowledge.store" target="_blank" rel="noopener">Knowledge Store portal</a>.

<img class=" wp-image-850 aligncenter" src="/images/devisland/article8/assets/cortanaskills441.png?w=300" alt="" width="595" height="319" />

You can read more about it in the official <a href="https://docs.microsoft.com/en-us/cortana/skills/publish-skill#discovery-and-management" target="_blank" rel="noopener">documentation</a> in "Discovery and Management" section.
<h3><strong>Wrapping up</strong></h3>
With Cortana smart assistant you can communicate with services in simple and "human-friendly" way by voice or text. You can also see that whole topic is complex so I wanted to create this article to make it easier to start using Cortana Skills Kit.