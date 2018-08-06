<img class="aligncenter wp-image-35" src="https://devislandblog.files.wordpress.com/2018/04/botframeworkcover.png?w=775" alt="" width="440" height="330" />
<h3><strong>Short introduction</strong></h3>
Microsoft Bot Framework enables developers to create intelligent applications to communicate with users. In the first part I described some key concepts connected with Bot Framework and in this article I would like to discuss the source code of created Bot.
<h3><strong>Basic bot source code</strong></h3>
During Bot creation in Microsft Azure portal (described in part 1) I chose "Basic C#" template for Bot. Now lets discover how source code of such Bot looks like.
<p style="text-align:center;"><img class="alignnone size-medium wp-image-100" src="https://devislandblog.files.wordpress.com/2018/04/botf1.png?w=287" alt="" width="287" height="300" /></p>
&nbsp;

Download source code of your Bot from Azure Portal:
<p style="text-align:center;"><img class="aligncenter wp-image-104 " src="https://devislandblog.files.wordpress.com/2018/04/botf2.png?w=775" alt="" width="545" height="194" /></p>
Open project in Visual Studio:

<img class=" wp-image-106 aligncenter" src="https://devislandblog.files.wordpress.com/2018/04/botf3.png?w=255" alt="" width="265" height="312" />

&nbsp;

First class we will review is called "MessagesController" and its located in "Controllers" folder:
<p style="text-align:center;"><img class="alignnone size-medium wp-image-121" src="https://devislandblog.files.wordpress.com/2018/04/botf6.png?w=273" alt="" width="273" height="300" /></p>


[code language="csharp"]
 [BotAuthentication]
    public class MessagesController : ApiController
    {
        /// 
<summary>
        /// POST: api/Messages
        /// receive a message from a user and send replies
        /// </summary>

        /// <param name="activity"></param>
        [ResponseType(typeof(void))]
        public virtual async Task<HttpResponseMessage> Post([FromBody] Activity activity)
        {
            // check if activity is of type message
            if (activity != null && activity.GetActivityType() == ActivityTypes.Message)
            {
                await Conversation.SendAsync(activity, () => new EchoDialog());
            }
            else
            {
                HandleSystemMessage(activity);
            }
            return new HttpResponseMessage(System.Net.HttpStatusCode.Accepted);
        }

        private Activity HandleSystemMessage(Activity message)
        {
            if (message.Type == ActivityTypes.DeleteUserData)
            {
                // Implement user deletion here
                // If we handle user deletion, return a real message
            }
            else if (message.Type == ActivityTypes.ConversationUpdate)
            {
                // Handle conversation state changes, like members being added and removed
                // Use Activity.MembersAdded and Activity.MembersRemoved and Activity.Action for info
                // Not available in all channels
            }
            else if (message.Type == ActivityTypes.ContactRelationUpdate)
            {
                // Handle add/remove from contact lists
                // Activity.From + Activity.Action represent what happened
            }
            else if (message.Type == ActivityTypes.Typing)
            {
                // Handle knowing tha the user is typing
            }
            else if (message.Type == ActivityTypes.Ping)
            {
            }

            return null;
        }
    }
[/code]

<strong>BotAuthentication</strong>

It is important to secure access to the Bot. To prevent unauthorized access "MessagesController" class has "BotAuthentication" attribute. To access Bot you have to provide "MicrosoftAppId" and "MicrosoftAppPassword". You can find them in Azure portal in your Bot settings:

<img class=" wp-image-132 aligncenter" src="https://devislandblog.files.wordpress.com/2018/04/botf81.png?w=158" alt="" width="166" height="316" />
<p style="text-align:center;"><img class="alignnone size-medium wp-image-133" src="https://devislandblog.files.wordpress.com/2018/04/botf91.png?w=300" alt="" width="300" height="62" /></p>
Now with AppId and Password you can test connection to your Bot from Bot Emulator:
<p style="text-align:center;"><img class="alignnone wp-image-135" src="https://devislandblog.files.wordpress.com/2018/04/botf7.png?w=300" alt="" width="308" height="156" /></p>
You have to paste AppId and Password in "Web.config" file:

<img class=" wp-image-137 aligncenter" src="https://devislandblog.files.wordpress.com/2018/04/botf10.png?w=300" alt="" width="415" height="61" />

<b>Activity</b>

An Activity is the basic communication type for the Bot Framework 3.0 protocol. Activity object is used to pass information back and forth between bot and channel (user). The most popular Activity is message (which represents a communication between Bot and user) but there are others like typing (when user or bot is typing the response). You can read more <a href="https://docs.microsoft.com/en-us/azure/bot-service/dotnet/bot-builder-dotnet-activities" target="_blank" rel="noopener">here.</a>

<b>Conversation</b>

Conversation represents channel between Bot and user through which new activities (like messages) are reported. You can model conversation flow

<b>EchoDialog</b>

Second class we will review is called "EchoDialog" and its located in "Dialogs" folder:
<p style="text-align:center;"><img class="alignnone size-medium wp-image-108" src="https://devislandblog.files.wordpress.com/2018/04/botf4.png?w=300" alt="" width="300" height="258" /></p>


[code language="csharp"]

  [Serializable]
    public class EchoDialog : IDialog<object>
    {
        protected int count = 1;

        public async Task StartAsync(IDialogContext context)
        {
            context.Wait(MessageReceivedAsync);
        }

        public async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> argument)
        {
            var message = await argument;

            if (message.Text == "reset")
            {
                PromptDialog.Confirm(
                    context,
                    AfterResetAsync,
                    "Are you sure you want to reset the count?",
                    "Didn't get that!",
                    promptStyle: PromptStyle.Auto);
            }
            else
            {
                await context.PostAsync($"{this.count++}: You said {message.Text}");
                context.Wait(MessageReceivedAsync);
            }
        }

        public async Task AfterResetAsync(IDialogContext context, IAwaitable<bool> argument)
        {
            var confirm = await argument;
            if (confirm)
            {
                this.count = 1;
                await context.PostAsync("Reset count.");
            }
            else
            {
                await context.PostAsync("Did not reset count.");
            }
            context.Wait(MessageReceivedAsync);
        }
    }

[/code]

<strong>IDialog</strong>

Interface from Microsoft.Bot.Builder.Dialogs namespace. Each C# class which extends this interface is an abstraction that encapsulates its own state. Use dialogs to model a conversation and manage conversation flow.

It is worth to say few more words about Dialogs here.

Like apps and websites, bots have a UI, but it is made up of dialogs, rather than screens. Dialogs may or may not have graphical interfaces. They may contain buttons, text, and other elements, or be entirely speech-based. Dialogs also contain actions to perform tasks such as invoking other dialogs or processing user input.

Below there is a diagram to show comparsion between the screen flow of a traditional application compared to the dialog flow of a bot:

<img class=" wp-image-118 aligncenter" src="https://devislandblog.files.wordpress.com/2018/04/botf5.png?w=300" alt="" width="462" height="203" />

When one dialog invokes another, the Bot Builder adds the new dialog to the top of the dialog stack. The dialog that is on top of the stack is in control of the conversation.

<strong>IDialogContext</strong>

IDialogContext interface represents context of current conversation between user and Bot. Below you can see inheritance diagram for IDialogContext:
<p style="text-align:center;"><img class="alignnone wp-image-146" src="https://devislandblog.files.wordpress.com/2018/04/botf11.png" alt="" width="217" height="233" /></p>
Please note that "MessageReceivedAsync" method has two parameters: context and argument. It means that for each new message sent to Bot there is specific context and argument assigned.

&nbsp;
<h3><strong>Wrapping up</strong></h3>
Microsoft Bot Framework enables creating smart applications which can communicate with users with natural conversation flow. Developers are able to write specific functionality of bots using Bot Builder SDK available for NET C# and Node.js and host their bots on Microsoft Azure platform. If you would like to read more please refer to official <a href="https://docs.microsoft.com/en-us/azure/bot-service/">documentation.</a>

In the next part I will show how to connect Bot to different channels like Skype and how to communicate with created Bot from Xamarin application.