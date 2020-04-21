---
title: "Motion detector - automatic SMS alerts with Windows IoT Core and Microsoft Azure cloud"
excerpt: "In this article I would like to present how to detect motion with Raspberry Pi 2 device running Windows IoT Core system, connected to Microsoft Azure cloud."
header:
  image: /images/cloudyofthings/article1/assets/CloudyOfThingsArticle1.png
---

![Image](/images/cloudyofthings/article1/assets/CloudyOfThingsArticle1.png?raw=true)

## Use case

In this article I would like to present how to detect motion with Raspberry Pi 2 device running Windows IoT Core system, connected to Microsoft Azure cloud. Once motion is detected, there is SMS sent to my cell phone. Once you read this article you will have knowledge about connecting IoT Core device with Azure cloud, which sensors should be used and how to properly integrate services in Azure.

If you do not not how to setup Windows IoT Core device, please refer to my blog post article [here](https://devislandblog.wordpress.com/2018/08/14/windows-10-iot-core-device-connected-to-the-microsoft-azure-cloud/)

## Solution

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetector.png?raw=true" alt="Solution diagram"/>
</p>

Raspberry Pi 2 device with Windows IoT Core system is connected to the Azure IoT Hub. This enables to send messages from device to the cloud. In this example motion sensor (standard PIR sensor HC-SR501) is used and once it detects motion, information is sent to the Azure. At the end I receive SMS notification. 

Below there is flow presented:
1. Motion sensor connected to the Raspberry Pi detects motion
2. Raspberry Pi sends information to the Azure IoT Hub
3. Azure Stream Analytics detects new data from Azure IoT Hub (which is input for the Stream Analytics)
4. Stream Analytics calls Azure Function App (as its output for the Stream Analytics)
5. Azure Function App calls Twilio API to send SMS notification

Here is how motion sensor should be connected:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorSchema.png?raw=true" alt="Solution diagram"/>
</p>

## Code and Configuration

In this section you will find how to create solution described above.

### Motion Detector UWP application for Windows IoT Core

To detect motion from the sensor connected to the Raspberry Pi I wrote a Universal Windows Platform application. Whole source code you can find on my GitHub [here](https://github.com/Daniel-Krzyczkowski/WindowsIoT-AzureIoT/tree/master/WindowsIoTCore/MotionDetector)

The most important part is in the MainPage class. We have to setup Gpio pin correctly. We have to set DriveMode to "Input" so we would like to receive signal from the motion sensor. Then we hava ValueChanged event raised once there is edge change detected. If its "RisingEdge" it means that motion was detected - then we are using AzureIoTHubService instance to send this information to the Azure. AzureIoTHubService source code is presented below.

```csharp
 public sealed partial class MainPage : Page
    {

        private GpioController gpio;
        private GpioPin sensor;
        private AzureIoTHubService _azureIoTHubService;

        public MainPage()
        {
            this.InitializeComponent();
            this.InitializeComponent();

            InitGPIO();
            InitAzureIoTHub();
        }

        private void InitAzureIoTHub()
        {
            _azureIoTHubService = new AzureIoTHubService();
        }

        private void InitGPIO()
        {
            gpio = GpioController.GetDefault();
            if (gpio == null)
                return;
            GpioStatus.Text = "Gpio initialized";

            sensor = gpio.OpenPin(16);
            sensor.SetDriveMode(GpioPinDriveMode.Input);
            sensor.ValueChanged += Sensor_ValueChanged;
        }

        private async void Sensor_ValueChanged(GpioPin sender, GpioPinValueChangedEventArgs args)
        {
            if (args.Edge == GpioPinEdge.RisingEdge)
            {
                System.Diagnostics.Debug.WriteLine("MOTION DETECTED");

                await _azureIoTHubService.SendDataToAzure(new Model.MotionEvent());

                await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal,
                 () =>
                 {
                     textPlaceHolder.Text = "MOTION DETECTED";
                 });
            }

            else
            {
                System.Diagnostics.Debug.WriteLine("NO MOTION DETECTED");
                await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal,
                 () =>
                 {
                     textPlaceHolder.Text = "NO MOTION DETECTED";
                 });
            }
        }
    }
```

AzureIoTHubService class. We are sending MotionEvent with fake room number where motion was detected:

```csharp
    public class AzureIoTHubService
    {
        private DeviceClient _deviceClient;

        public AzureIoTHubService()
        {
            _deviceClient = DeviceClient.CreateFromConnectionString("<<Azure IoT Hub Connection String>>", TransportType.Http1);
        }

        public async Task<bool> SendDataToAzure(MotionEvent motionEvent)
        {
            var messageString = JsonConvert.SerializeObject(motionEvent);
            var message = new Message(Encoding.ASCII.GetBytes(messageString));

            await _deviceClient.SendEventAsync(message);

            Debug.WriteLine("{0} > Sending telemetry: {1}", DateTime.Now, messageString);
            return true;
        }
    }
```

### Microsoft Azure services configuration

In this section you will find which services were used in the Azure cloud and how to configure them. I assume that you have active Azure subscription.

### Azure IoT Hub

Sign in to the portal and search for IoT Hub:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure1.PNG?raw=true" alt="MotionDetectorAzure1.png"/>
</p>

Then type the name of the resource group (we will have all Azure services collected here) and name of your IoT Hub:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure2.PNG?raw=true" alt="MotionDetectorAzure2.png"/>
</p>

You can select either basic or free tier for tests. In the summary blade click create:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure3.PNG?raw=true" alt="MotionDetectorAzure3.png"/>
</p>

Open IoT devices tab and create new device - type the name of registered device (for instane MyRaspberryPi):

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure4.PNG?raw=true" alt="MotionDetectorAzure4.png"/>
</p>

Once device is displayed on the list, click it and copy connection string with primary key:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure5.PNG?raw=true" alt="MotionDetectorAzure5.png"/>
</p>

### Azure Stream Analytics Job

##### IMPORTANT - use Stream Analytics for the further data analysis but you can also use simple solution with IoTHubTrigger presented further in the article.

Now we have to create Stream Analytics Job to get data from the Azure IoT Hub and pass it to the Azure Function. In the Azure portal search for Stream Analytics Job. Use the same resource group you created for the Azure IoT Hub. Type the name (feel free here) and do not change rest of proposed configuration:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure6.PNG?raw=true" alt="MotionDetectorAzure6.png"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure7.PNG?raw=true" alt="MotionDetectorAzure7.png"/>
</p>

We will configure Stream Analytics later in the article.

### Azure Function App

We want to have Azure Function which will send SMS notifications using Twilio API. In the Azure portal search for Function App. Use proposed configuration but remember to create it in the same resource group where Azure IoT Hub and Stream Analytics were created:


<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure9.PNG?raw=true" alt="MotionDetectorAzure9.png"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure10.PNG?raw=true" alt="MotionDetectorAzure10.png"/>
</p>

Create HTTP Trigger Function App. Below I present the code you should use:

```csharp
public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger logger)
{
    logger.LogInformation("C# HTTP trigger function processed a request.");

    string content = await req.Content.ReadAsStringAsync();

    logger.LogInformation("C# HTTP trigger function processed a request: " + content);

    dynamic array = Newtonsoft.Json.JsonConvert.DeserializeObject(content);

    foreach(var json in array)
    {
        logger.LogInformation($"Body: room number={json.RoomNumber}; datetime={json.EventProcessedUtcTime}; device={json.IoTHub.ConnectionDeviceId}");
    }
    
    string accountSid = ConfigurationManager.AppSettings["TwilioAccountSid"];
    string authToken = ConfigurationManager.AppSettings["TwilioAuthToken"];
    string fromNumber = ConfigurationManager.AppSettings["FromNumber"];
    string toNumber = ConfigurationManager.AppSettings["ToNumber"];

        TwilioClient.Init(accountSid, authToken);

        var message = MessageResource.Create(
            body: "MOTION DETECTED",
            from: new PhoneNumber(fromNumber),
            to: new PhoneNumber(toNumber)
        );

    return req.CreateResponse(HttpStatusCode.OK);
}
```

Values for TwlioClient are taken from the Function App Settings. You should have active Twilio account - its free and you can setup test number from which SMS will be sent for free. You can find full instruction [here](https://www.twilio.com/try-twilio).
Once you setup Twilio test account you should obtaind TwilioAccountSid, TwilioAuthToken, FromNumber, ToNumber values. Copy them to the Function App settings in the Azure portal.

One important thing here - you should add project.json file to add two NuGet packages. Below I present project.json file content:

```csharp
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Newtonsoft.Json": "11.0.2",
        "Microsoft.Extensions.Logging.Abstractions": "2.1.1",
        "Twilio": "5.16.4"
      }
    }
   }
}
```

### Azure Function IoTHubTrigger

##### Here I would like to say thanks to [Stefan Wick](https://github.com/Daniel-Krzyczkowski/WindowsIoTCore/issues/1) from Microsoft for a great hint!

As mentioned in the previous section with Azure Stream Analytics you can also use IoTHubTrigger to process the messages sent from the device to the Azure IoT Hub. Below I presented the steps how to configure IoTHubTrigger.

Open Functions blade and select "+" button to add new function:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure15_2.PNG?raw=true" alt="MotionDetectorAzure15_2.png"/>
</p>

Select "Event Hub Trigger C#" template:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure16.PNG?raw=true" alt="MotionDetectorAzure16.png"/>
</p>

Type the name: "MotionSensorDataTrigger" and in the "Event Hub connection" section click "New":
<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure17.PNG?raw=true" alt="MotionDetectorAzure17.png"/>
</p>

Select "IoT Hub" tab then select previously created IoT Hub and endpoint:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure18.PNG?raw=true" alt="MotionDetectorAzure18.png"/>
</p>

That's it! Now you can paste the code responsible for sending SMS messages there from the previous function. Check the logs to see what messages are received:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure19.PNG?raw=true" alt="MotionDetectorAzure19.png"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure20.PNG?raw=true" alt="MotionDetectorAzure20.png"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure21.PNG?raw=true" alt="MotionDetectorAzure21.png"/>
</p>


### Azure Stream Analytics Job input and output configuration

Now get back to the Stream Analytics Job in the Azure portal and click input tab. You should select previously created IoT Hub as an input. Then got to the output tab and select above Azure Function App. Last step is to update Query with below code:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure8.PNG?raw=true" alt="MotionDetectorAzure8.png"/>
</p>

```csharp
SELECT
    *
INTO
    output
FROM
    input
```

We want to take whole data from the Azure IoT Hub and pass it to the Function App. 

Important - remember to start Stream Analytics Job.

## Final project

Once you launch the UWP application on the Raspberry Pi device you should receive SMS if motion was detected.
This is my final project:

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure11.png?raw=true" alt="MotionDetectorAzure11.png"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure12.JPG?raw=true" alt="MotionDetectorAzure12.JPG"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure13.JPG?raw=true" alt="MotionDetectorAzure13.JPG"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure14.JPG?raw=true" alt="MotionDetectorAzure14.JPG"/>
</p>


<p align="center">
  <img src="/images/cloudyofthings/article1/assets/MotionDetectorAzure15.PNG?raw=true" alt="MotionDetectorAzure15.png"/>
</p>



[Back](https://daniel-krzyczkowski.github.io/)
