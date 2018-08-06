
Short introduction
Microsoft Azure Notification Hub is a service which enables developers sending push notifications to different platforms like iOS, Android or Windows. Most of mobile applications are integrated with backend and in this article I would like to present how to integrate Notification Hub with ASP .NET Core Web API.



Architecutre
Before I present some code I would like to focus on architecture aspect. How exactly Azure Notification Hub works and how it is integrated with applications.



Azure Notification Hub is a kind of proxy which pass push notification via different providers. Let's discuss the whole flow form the diagram presented above.

ASP .NET Core Web API application sends push notification request to Azure Notification Hub
Azure Notification Hub is connected with different Push Notification Services:
Apple Push Notification Service - iOS
Firebase Cloud Messaging (previously called Google Cloud Messaging) - Android
Windows Notification Service - Windows
Each Push Notification Service delivers push notification to specific platform
As you can see Microsoft Azure Notification Hub is a proxy here which makes it easier to send push notifications.



Azure Notification Hub configuration
In this section we will go through configuration steps in Azure portal.

Search for new resource in Marketplace. Type "Notification hub":



Select "Notification Hub" and click "Create" button. In the next blade you have to provide few details:

Name of Notification Hub
Namespace (you can have one one namespace with more than one notification hubs)
Location of the hub server
Resource group in which Notification Hub will be created
Azure subscription
Pricing tier - for now select "FREE"


Click "Create" button. Once Hub is created it becomes visible in resource group:



Click on it to display details:



As you can see in "FREE" tier there can be 500 devices registered in the Hub and 1 million pushes can be sent. On the left bar you can notice "Notification settings" section:



Azure Notification Hub can be integrated with Apple, Google, Windows, Amazon and Baidu. In my sample I will concentrate on push notifications for iOS, Android and Windows platforms.

Android

To setup Azure Notification Hub  with Android platform you have to generate "API Key" which should be generated in Google Developers Console. Please follow instructions under this link.



Once you paste the key, click "Save" button.

iOS

Azure Notification Hub setup for iOS requires .p12 certificate with password generated on Mac computer. Instructions are available under this link. Important note here - if you want to test push notifications in debug mode, select "Sandbox" in "Application Mode" below.



Windows

Configuration for Universal Windows Platforms apps (UWP) requires "Package SID" and "Security Key" which can be generated in Windows Developer portal. Follow this instructions.





ASP .NET Core Web API integration with Azure Notification Hub
Our goal is to manage push notifications from the backend side. That is why ASP .NET Core Web API has to be integrated with Azure Notification Hub. Below I present diagram which shows communication flow between them:



Above diagram presents process of push notification delivery process:

Each application (Android, iOS and Windows) calls specific Push Notification Service to get push handle - unique token assigned to device
Then each application (Android, iOS, Windows) calls backend (ASP .NET Core Web API) to obtain registration ID from Azure Notification Hub
Then each application (Android, iOS and Windows) sends push handle obtained from Push Notification Service and registration ID obtained in step above to backend
Once applications are registered they can receive push notifications
Backend sends push notification request to Azure Notification Hub which then distribute pushes to each Push Notification Service
At the end each application (Android, iOS and Windows) receive push notification from dedicated Push Notification Service
Project setup

Once everything is configured in Azure portal it is time to update ASP .NET Core Web API project with push notifications functionality. Create new blank project (or open existing):



Add new Controller called "PushNotificationsController":



Now add one more folder called " Configuration" and add new class inside it called "NotificationHubConfiguration":



Source code should look like this:

[code language="csharp"]
   public class NotificationHubConfiguration
    {
        public string ConnectionString { get; set; }
        public string HubName { get; set; }
    }
[/code]
Now open "appsettings.json" file and add new section called "NotificationHub":

[code language="csharp"]
{
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  },
  "NotificationHub": {
    "ConnectionString": "Endpoint...",
    "HubName": "DevIslandNotificationHub"
  }
}

[/code]
ConnectionString and HubName values can be obtain in Azure portal. HubName is just a name of notification hub set in portal and ConnectionString value is obtained from "Access Policies" section. "DefaultFullSharedAccessSignature" connection string should be copied:



Now open "Startup.cs" class and replace "Startup" method with below code:

[code language="csharp"]
     public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }
[/code]
Now replace "ConfigureServices" with below code. Note that I mapped Notification Hub configuration from "appsettings.json" file to class called "NotificationHubConfiguration":

[code language="csharp"]
 public void ConfigureServices(IServiceCollection services)
        {
            services.Configure<NotificationHubConfiguration>(Configuration.GetSection("NotificationHub"));
            services.AddMvc();
        }
[/code]


To integrate ASP .NET Core Web API application with Azure Notification Hub NuGet package is required. Add "Microsoft.Azure.NotificationHubs" package. At the moment of writing this article package was in preview so "Include prerelease" checkbox has to be enabled:



Once package is added, create new folder called "NotificationHubs" and add "NotificationHubProxy" class, "DeviceRegistration" class, "HubResponse" class, "Notification" class and "MobilePlatform" enumeration:



Here is how "DeviceRegistration" class should look like. Lets discuss its fields.

MobilePlatform - enum which indicated which Push Notification Service (Google - gmc, Apple - apns, Windows - wns) should be used to deliver notification
Handle - token obtained from Push Notification Service
Tags - there is possibility to tag notifications. Then only registrations that contain the specified tag receive the notification
[code language="csharp"]
 public class DeviceRegistration
    {
        [JsonConverter(typeof(StringEnumConverter))]
        public MobilePlatform Platform { get; set; }
        public string Handle { get; set; }
        public string[] Tags { get; set; }
    }
[/code]


Here is how "HubResponse" class should look like. This class is responsible for wrapping result of operations invoked on Azure Notification Hub.

[code language="csharp"]
 public sealed class HubResponse<TResult> : HubResponse where TResult : class
    {
        public TResult Result { get; set; }

        public HubResponse()
        {
        }

        public new HubResponse<TResult> SetAsFailureResponse()
        {
            base.SetAsFailureResponse();
            return this;
        }

        public new HubResponse<TResult> AddErrorMessage(string errorMessage)
        {
            base.AddErrorMessage(errorMessage);
            return this;
        }
    }

    public class HubResponse
    {
        private bool _forcedFailedResponse;

        public bool CompletedWithSuccess => !ErrorMessages.Any() && !_forcedFailedResponse;
        public IList<string> ErrorMessages { get; private set; }

        public HubResponse()
        {
            ErrorMessages = new List<string>();
        }

        public HubResponse SetAsFailureResponse()
        {
            _forcedFailedResponse = true;
            return this;
        }

        public HubResponse AddErrorMessage(string errorMessage)
        {
            ErrorMessages.Add(errorMessage);
            return this;
        }

        public string FormattedErrorMessages => ErrorMessages.Any()
            ? ErrorMessages.Aggregate((prev, current) => prev + Environment.NewLine + current)
            : string.Empty;
    }
[/code]


Here is how "MobilePlatform" enum should look like. With this enum we specify Push Notification Service used to deliver notification:

wns - Windows Notification Service
apns - Apple Push Notifications Service
gcm - Google Cloud Messaging


[code language="csharp"]
    public enum MobilePlatform
    {
        wns, apns, gcm
    }
[/code]


Here is how "Notification" class should look like. With this class notification objects are created with specific content and configuration.

[code language="csharp"]
    public class Notification : DeviceRegistration
    {
        public string Content { get; set; }
    }
[/code]


Here is how "NotificationHubProxy" class should look like:

[code language="csharp"]
 public class NotificationHubProxy
    {
        private NotificationHubConfiguration _configuration;
        private NotificationHubClient _hubClient;

        public NotificationHubProxy(NotificationHubConfiguration configuration)
        {
            _configuration = configuration;
            _hubClient = NotificationHubClient.CreateClientFromConnectionString(_configuration.ConnectionString, _configuration.HubName);
        }

        /// 
        /// 

<summary>
        /// Get registration ID from Azure Notification Hub
        /// </summary>


        public async Task<string> CreateRegistrationId()
        {
            return await _hubClient.CreateRegistrationIdAsync();
        }

        /// 
        /// 

<summary>
        /// Delete registration ID from Azure Notification Hub
        /// </summary>


        /// <param name="registrationId"></param>
        public async Task DeleteRegistration(string registrationId)
        {
            await _hubClient.DeleteRegistrationAsync(registrationId);
        }

        /// 
        /// 

<summary>
        /// Register device to receive push notifications. 
        /// Registration ID ontained from Azure Notification Hub has to be provided
        /// Then basing on platform (Android, iOS or Windows) specific
        /// handle (token) obtained from Push Notification Service has to be provided
        /// </summary>


        /// <param name="id"></param>
        /// <param name="deviceUpdate"></param>
        /// <returns></returns>
        public async Task<HubResponse> RegisterForPushNotifications(string id, DeviceRegistration deviceUpdate)
        {
            RegistrationDescription registrationDescription = null;

            switch (deviceUpdate.Platform)
            {
                case MobilePlatform.wns:
                    registrationDescription = new WindowsRegistrationDescription(deviceUpdate.Handle);
                    break;
                case MobilePlatform.apns:
                    registrationDescription = new AppleRegistrationDescription(deviceUpdate.Handle);
                    break;
                case MobilePlatform.gcm:
                    registrationDescription = new GcmRegistrationDescription(deviceUpdate.Handle);
                    break;
                default:
                    return new HubResponse().AddErrorMessage("Please provide correct platform notification service name.");
            }

            registrationDescription.RegistrationId = id;
            if (deviceUpdate.Tags != null)
                registrationDescription.Tags = new HashSet<string>(deviceUpdate.Tags);

            try
            {
                await _hubClient.CreateOrUpdateRegistrationAsync(registrationDescription);
                return new HubResponse();
            }
            catch (MessagingException)
            {
                return new HubResponse().AddErrorMessage("Registration failed because of HttpStatusCode.Gone. PLease register once again.");
            }
        }

        /// 
        /// 

<summary>
        /// Send push notification to specific platform (Android, iOS or Windows)
        /// </summary>


        /// <param name="newNotification"></param>
        /// <returns></returns>
        public async Task<HubResponse<NotificationOutcome>> SendNotification(Notification newNotification)
        {
            try
            {
                NotificationOutcome outcome = null;

                switch (newNotification.Platform)
                {
                    case MobilePlatform.wns:
                        if (newNotification.Tags == null)
                            outcome = await _hubClient.SendWindowsNativeNotificationAsync(newNotification.Content);
                        else
                            outcome = await _hubClient.SendWindowsNativeNotificationAsync(newNotification.Content, newNotification.Tags);
                        break;
                    case MobilePlatform.apns:
                        if (newNotification.Tags == null)
                            outcome = await _hubClient.SendAppleNativeNotificationAsync(newNotification.Content);
                        else
                            outcome = await _hubClient.SendAppleNativeNotificationAsync(newNotification.Content, newNotification.Tags);
                        break;
                    case MobilePlatform.gcm:
                        if (newNotification.Tags == null)
                            outcome = await _hubClient.SendGcmNativeNotificationAsync(newNotification.Content);
                        else
                            outcome = await _hubClient.SendGcmNativeNotificationAsync(newNotification.Content, newNotification.Tags);
                        break;
                }

                if (outcome != null)
                {
                    if (!((outcome.State == NotificationOutcomeState.Abandoned) ||
                        (outcome.State == NotificationOutcomeState.Unknown)))
                        return new HubResponse<NotificationOutcome>();
                }

                return new HubResponse<NotificationOutcome>().SetAsFailureResponse().AddErrorMessage("Notification was not sent due to issue. Please send again.");
            }

            catch (MessagingException ex)
            {
                return new HubResponse<NotificationOutcome>().SetAsFailureResponse().AddErrorMessage(ex.Message);
            }

            catch (ArgumentException ex)
            {
                return new HubResponse<NotificationOutcome>().SetAsFailureResponse().AddErrorMessage(ex.Message);
            }
        }
    }
[/code]


Last step is to create dedicated Controller for handling push notification requests:

[code language="csharp"]

    /// 
    /// 

<summary>
    /// Anonymous access is only for testing purposes
    /// Remember to enable authentication
    /// </summary>



    [AllowAnonymous]
    [Produces("application/json")]
    [Route("api/notifications")]
    public class PushNotificationsController : Controller
    {
        private NotificationHubProxy _notificationHubProxy;

        public PushNotificationsController(IOptions<NotificationHubConfiguration> standardNotificationHubConfiguration)
        {
            _notificationHubProxy = new NotificationHubProxy(standardNotificationHubConfiguration.Value);
        }

        /// 
        /// 

<summary>
        /// Get registration ID
        /// </summary>


        /// <returns></returns>
        [HttpGet("register")]
        public async Task<IActionResult> CreatePushRegistrationId()
        {
            var registrationId = await _notificationHubProxy.CreateRegistrationId();
            return Ok(registrationId);
        }

        /// 
        /// 

<summary>
        /// Delete registration ID and unregister from receiving push notifications
        /// </summary>


        /// <param name="registrationId"></param>
        /// <returns></returns>
        [HttpDelete("unregister/{registrationId}")]
        public async Task<IActionResult> UnregisterFromNotifications(string registrationId)
        {
            await _notificationHubProxy.DeleteRegistration(registrationId);
            return Ok();
        }

        /// 
        /// 

<summary>
        /// Register to receive push notifications
        /// </summary>


        /// <param name="id"></param>
        /// <param name="deviceUpdate"></param>
        /// <returns></returns>
        [HttpPut("enable/{id}")]
        public async Task<IActionResult> RegisterForPushNotifications(string id, [FromBody] DeviceRegistration deviceUpdate)
        {
            HubResponse registrationResult = await _notificationHubProxy.RegisterForPushNotifications(id, deviceUpdate);

            if (registrationResult.CompletedWithSuccess)
                return Ok();

            return BadRequest("An error occurred while sending push notification: " + registrationResult.FormattedErrorMessages);
        }

        /// 
        /// 

<summary>
        /// Send push notification
        /// </summary>


        /// <param name="newNotification"></param>
        /// <returns></returns>
        [HttpPost("send")]
        public async Task<IActionResult> SendNotification([FromBody] Notification newNotification)
        {
            HubResponse<NotificationOutcome> pushDeliveryResult = await _notificationHubProxy.SendNotification(newNotification);

            if (pushDeliveryResult.CompletedWithSuccess)
                return Ok();

            return BadRequest("An error occurred while sending push notification: " + pushDeliveryResult.FormattedErrorMessages);
        }
    }
[/code]


Universal Windows App to test push notifications
I prepared UWP application to test push notifications functionality:



I call ASP .NET Core Web API backend from this application.

There are two important classes: "RestService" to call API and "MainViewModel" to keep application logic:

[code language="csharp"]
public class RestService
    {
        private HttpClient _httpClient;
        private readonly string _serviceAddress = "http://localhost:52978/api/notifications";
        public RestService()
        {
            _httpClient = new HttpClient();
        }

        public async Task<string> GetPushRegistrationId()
        {
            string registrationId = string.Empty;
            HttpResponseMessage response = await _httpClient.GetAsync(string.Concat(_serviceAddress, "/register"));
            if (response.IsSuccessStatusCode)
            {
                var jsonResponse = await response.Content.ReadAsStringAsync();
                registrationId = JsonConvert.DeserializeObject<string>(jsonResponse);
            }

            return registrationId;
        }

        public async Task<bool> UnregisterFromNotifications(string registrationId)
        {
            HttpResponseMessage response = await _httpClient.DeleteAsync(string.Concat(_serviceAddress, $"/unregister/{registrationId}"));
            if (response.IsSuccessStatusCode)
                return true;
            return false;
        }

        public async Task<bool> EnablePushNotifications(string id, DeviceRegistration deviceUpdate)
        {
            string registrationId = string.Empty;
            var content = new StringContent(JsonConvert.SerializeObject(deviceUpdate), Encoding.UTF8, "application/json");

            HttpResponseMessage response = await _httpClient.PutAsync(string.Concat(_serviceAddress, $"/enable/{id}"), content);
            if (response.IsSuccessStatusCode)
                return true;
            return false;
        }

        public async Task<bool> SendNotification(Notification newNotification)
        {
            var content = new StringContent(JsonConvert.SerializeObject(newNotification), Encoding.UTF8, "application/json");

            HttpResponseMessage response = await _httpClient.PostAsync(string.Concat(_serviceAddress, "/send"), content);
            if (response.IsSuccessStatusCode)
                return true;
            return false;
        }
    }
[/code]
[code language="csharp"]
public class MainViewModel : INotifyPropertyChanged
    {
        private string _status;
        private string _registrationId;
        private readonly RestService _restService;
        public string Status
        {
            get { return _status; }
            set
            {
                _status = value;
                OnPropertyChanged(nameof(Status));
            }
        }

        public string RegistrationId
        {
            get { return _registrationId; }
            set
            {
                _registrationId = value;
                OnPropertyChanged(nameof(_registrationId));
            }
        }

        public ICommand GetRegistrationIdCommand { get; private set; }

        public ICommand RegisterForPushNotifications { get; private set; }

        public ICommand UnregisterFromPushNotifications { get; private set; }

        public ICommand SendPushNotification { get; private set; }

        public MainViewModel()
        {
            _restService = new RestService();

            GetRegistrationIdCommand = new DelegateCommand(async (parameter) =>
            {
                RegistrationId = await _restService.GetPushRegistrationId();
                Status = "Registration ID obtained: " + RegistrationId;
            });

            RegisterForPushNotifications = new DelegateCommand(async(parameter) =>
            {
                var handle = await InitNotificationsAsync();

                var deviceUpdate = new DeviceRegistration()
                {
                    Handle = handle,
                    Platform = MobilePlatform.wns
                };

                var result = await _restService.EnablePushNotifications(RegistrationId, deviceUpdate);
                if (result)
                    Status = "Successfuly enabled push notifications";
                else
                    Status = "Failed to enable push notifications";
            });

            UnregisterFromPushNotifications = new DelegateCommand(async (parameter) =>
            {
                var result = await _restService.UnregisterFromNotifications(RegistrationId);
                if (result)
                    Status = "Successfuly unregistered from push notifications";
                else
                    Status = "Failed to unregister from push notifications";
            });

            SendPushNotification = new DelegateCommand(async(parameter) =>
            {
                var notification = new PushNotificationsApp.UWP.Model.Notification()
                {
                    Content = "<?xml version=\"1.0\" encoding=\"utf-8\"?><toast><visual><binding template = \"ToastText01\"><text id = \"1\"> Test message </text></binding></visual></toast>"
                };

                var result = await _restService.SendNotification(notification);
                if (result)
                    Status = "Successfuly sent push notifications";
                else
                    Status = "Failed to send push notifications";
            });
        }

        private async Task<string> InitNotificationsAsync()
        {
            // Get a channel URI from WNS.
            var channel = await PushNotificationChannelManager
                .CreatePushNotificationChannelForApplicationAsync();

            channel.PushNotificationReceived += Channel_PushNotificationReceived;

            return channel.Uri;
        }

        private void Channel_PushNotificationReceived(PushNotificationChannel sender, PushNotificationReceivedEventArgs args)
        {
            ShowToastNotification(args.ToastNotification);
        }

        private void ShowToastNotification(ToastNotification notification)
        {
            ToastNotifier ToastNotifier = ToastNotificationManager.CreateToastNotifier();
            notification.ExpirationTime = DateTime.Now.AddSeconds(4);
            ToastNotifier.Show(notification);
        }

        public event PropertyChangedEventHandler PropertyChanged;
        protected void OnPropertyChanged(string name)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
        }
    }
[/code]


Wrapping up
In this article I explained what Microsoft Azure Notification Hub is and how it supports sending push notifications to different platforms - Android, Windows and iOS. I also presented how to integrate ASP .NET Core Web API backend with Microsoft Azure Notification Hub. Source code for UWP and backend applications is available on my GitHub:

PushNotificationsWebApi
AzurePushNotificationHubApp