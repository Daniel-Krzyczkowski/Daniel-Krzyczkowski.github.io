# Motion detector - automatic alerts with Windows IoT Core and Microsoft Azure cloud


![Image](https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/mainassets/CloudyOfThings.png?raw=true)

## Use case
In this article I would like to present how to detect motion with Raspberry Pi 2 device running Windows IoT Core system, connected to Microsoft Azure cloud. Once motion is detected, there is SMS sent to my cell phone. Once you read this article you will have knowledge about connecting IoT Core device with Azure cloud, which sensors should be used and how to properly integrate services in Azure.

## Solution

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article1/assets/MotionDetector.png?raw=true" alt="Solution diagram"/>
</p>

Raspberry Pi 2 device with Windows IoT Core system is connected to the Azure IoT Hub. This enables to send messages from device to the cloud. In this example motion sensor is used and once it detects motion, information is sent to the Azure. At the end I receive SMS notification. 

Below there is flow presented:
1. Motion sensor connected to the Raspberry Pi detects motion
2. Raspberry Pi sends information to the Azure IoT Hub
3. Azure Stream Analytics detects new data from Azure IoT Hub (which is input for the Stream Analytics)
4. Stream Analytics calls Azure Function App (as its output for the Stream Analytics)
5. Azure Function App calls Twilio API to send SMS notification

Here is how motion sensor should be connected:

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article1/assets/MotionDetectorSchema.png?raw=true" alt="Solution diagram"/>
</p>

## Code and Configuration
In this section you will find how to create solution described above.

### Motion Detector UWP application for Windows IoT Core
To detect motion from the sensor connected to the Raspberry Pi I wrote a Universal Windows Platform application. Whole source code you can find on my GitHub [here](https://github.com/Daniel-Krzyczkowski/WindowsIoTCore/tree/master/MotionDetector)

The most important part is in the MainPage class. We have to setup Gpio pin correctly. We have to set DriveMode to "Input" so we would like to receive signal from the motion sensor. Then we hava ValueChanged event raised once there is edge change detected. If its "RisingEdge" it means that motion was detected - then we are using AzureIoTHubService instance to send this information to the Azure. AzureIoTHubService source code is presented below.

```
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

AzureIoTHubService class:

```
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
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article1/assets/MotionDetectorAzure1.PNG?raw=true" alt="MotionDetectorAzure1.png"/>
</p>

Then type the name of the resource group (we will have all Azure services collected here) and name of your IoT Hub:

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article1/assets/MotionDetectorAzure2.PNG?raw=true" alt="MotionDetectorAzure2.png"/>
</p>

You can select either basic or free tier for tests. In the summary blade click create:

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article1/assets/MotionDetectorAzure3.PNG?raw=true" alt="MotionDetectorAzure3.png"/>
</p>

Open IoT devices tab and create new device - type the name of registered device (for instane MyRaspberryPi):

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article1/assets/MotionDetectorAzure4.PNG?raw=true" alt="MotionDetectorAzure4.png"/>
</p>

Once device is displayed on the list, click it and copy connection string with primary key:

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article1/assets/MotionDetectorAzure5.PNG?raw=true" alt="MotionDetectorAzure5.png"/>
</p>

### Azure Stream Analytics Job
Now we have to create Stream Analytics Job to get data from the Azure IoT Hub and pass it to the Azure Function. In the Azure portal search for Stream Analytics Job. Use the same resource group you created for the Azure IoT Hub. Type the name (feel free here) and do not change rest of proposed configuration:

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article1/assets/MotionDetectorAzure6.PNG?raw=true" alt="MotionDetectorAzure6.png"/>
</p>

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article1/assets/MotionDetectorAzure7.PNG?raw=true" alt="MotionDetectorAzure7.png"/>
</p>

We will configure Stream Analytics later in the article.

### Azure Function App
Aaa

## Demo
Aaa

## Summary
Aaa

[Back](https://daniel-krzyczkowski.github.io/cloudyofthings/main/index)
