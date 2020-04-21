---
title: "Face detector - automatic e-mail alerts with Windows IoT Core and Microsoft Azure cloud"
excerpt: "In this article I would like to extend previous sample with face detection using camera connected to the IoT device and Microsoft Cognitive Serivces (Face API service)."
header:
  image: /images/cloudyofthings/article2/assets/CloudyOfThingsArticle2.png
---

![Image](/images/cloudyofthings/article2/assets/CloudyOfThingsArticle2.png?raw=true)

## Use case

In my previous article (you can find it [here](https://daniel-krzyczkowski.github.io/Motion-detector-automatic-SMS-alerts/)) I described how to connect motion sensor to the device with Windows IoT Core system and how to connect it with Microsoft Azure IoT Hub to send SMS alerts once motion is detected.

In this article I would like to extend previous sample with face detection using camera connected to the IoT device and Microsoft Cognitive Serivces (Face API service). Once motion is detected, camera takes picture which is analyzed with Cognitive Services Face API. At the end there is an email notification send with taken image and analyzis result using Azure Logic App.

## Solution

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetectorDiagram1.png?raw=true" alt="Solution diagram"/>
</p>

In the previous article I presented how to connect motion sensor with Raspberry Pi 2 device running Windows IoT Core system. This time we will add one more element - camera device connected to the USB port. Once motion is detected by sensor - photo is taken and analyzed with Microsoft Cognotive Services Face API. Then image together with analyzis result is sent to the Azure IoT Hub and then used with other components.

Below there is flow presented:
1. Motion sensor connected to the Raspberry Pi detects motion
2. Image is taken by connected camera device
3. Image is analyzed by Microsoft Cognitive Services Face API service
4. Analyzis result together with the taken image is sent to the Azure IoT Hub - image is saved in the Azure Blob Storage
5. Azure Function is triggered to pass analyzis result to the Azure Logic App
6. Azure Logic App receives analyzis result and retrieves taken image from the Azure Blob Storage
7. An e-mail is created with image attachment and then sent to the specific address

I also prepared UWP desktop application called "Face Identifier" to scan and identify people. This enables to register people so Cognitive Services Face API can then detect and analyze people from sent images. Application description is placed below in the article.

## Code and Configuration

In this section you will find how to create solution described above.

### Face Identifier UWP application - desktop app

Face Identifier UWP applicaton is connected with Microsoft Cognitive Services Face API. You can use it to take picture of your (or your friend) face. Then you can register new group and person in the Face API so once IoT device detects motion and takes image it can be analyzied and recognized. Application is presented below:


<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetectorApp1.png?raw=true" alt="Face identifier app"/>
</p>

As you can see I scaned my face and registered myself as "Daniel" in the "Home" group. Whole source code you can find on my GitHub [here.](https://github.com/Daniel-Krzyczkowski/UniversalWindowsPlatform/tree/master/FacesIdentifier)

The most important part is FaceRecognitionService class. There you can find all methods responsible creating group of known people, adding new person to the group (using image and name) or detecting it after training of Face API:

```csharp
    public class FaceRecognitionService
    {
        public event EventHandler<string> TrainingStatusChanged;
        private readonly FaceServiceClient _faceServiceClient;
        public FaceRecognitionService()
        {
            _faceServiceClient = new FaceServiceClient("<<Cognitive Services API Key>>", "https://westcentralus.api.cognitive.microsoft.com/face/v1.0");
        }

        public async Task CreatePersonGroup(string personGroupId, string groupName)
        {
            await _faceServiceClient.CreatePersonGroupAsync(personGroupId.ToLower(), groupName);
        }

        public async Task<Tuple<string, CreatePersonResult>> AddNewPersonToGroup(string personGroupId, string personName)
        {

            CreatePersonResult person = await _faceServiceClient.CreatePersonAsync(personGroupId, personName);

            return new Tuple<string, CreatePersonResult>(personGroupId, person);
        }

        public async Task<AddPersistedFaceResult> RegisterPerson(string personGroupId, CreatePersonResult person, Stream stream)
        {

            var addPersistedFaceResult = await _faceServiceClient.AddPersonFaceAsync(
                 personGroupId, person.PersonId, stream);
            return addPersistedFaceResult;
        }

        public async Task TrainPersonGroup(string personGroupId)
        {
            await _faceServiceClient.TrainPersonGroupAsync(personGroupId);
        }

        public async Task<TrainingStatus> VerifyTrainingStatus(string personGroupId)
        {
            TrainingStatus trainingStatus = null;
            while (true)
            {
                TrainingStatusChanged?.Invoke(this, "Training in progress...");
                trainingStatus = await _faceServiceClient.GetPersonGroupTrainingStatusAsync(personGroupId);

                if (trainingStatus.Status != Status.Running)
                {
                    break;
                }

                await Task.Delay(1000);
            }
            TrainingStatusChanged?.Invoke(this, "Training completed");
            return trainingStatus;
        }

        public async Task<string> VerifyFaceAgainstTraindedGroup(string personGroupId, Stream stream)
        {
            var faces = await _faceServiceClient.DetectAsync(stream);
            var faceIds = faces.Select(face => face.FaceId).ToArray();

            var results = await _faceServiceClient.IdentifyAsync(personGroupId, faceIds);
            foreach (var identifyResult in results)
            {
                if (identifyResult.Candidates.Length == 0)
                {
                    return "No one identified";
                }
                else
                {
                    // Get top 1 among all candidates returned
                    var candidateId = identifyResult.Candidates[0].PersonId;
                    var person = await _faceServiceClient.GetPersonAsync(personGroupId, candidateId);
                    return "Identified as: " + person.Name + " with face ID: " + identifyResult.FaceId;
                }
            }
            return "No one identified";
        }
    }
```

We are using FaceRecognitionService instance in the MainPage class so I encourage you to go through the source code of the app project.

### Face Detector UWP application for Windows IoT Core

Face Detector UWP application is created for IoT devices with Windows IoT Core system. Once motion is detected, camera connected to the device takes picture which is then analyzed by Microsoft Cognitive Services. Image and analyzis result are sent then to the Azure IoT Hub. Application is presented below:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetectorApp2.png?raw=true" alt="Face detector app"/>
</p>

Whole source code you can find on my GitHub [here.](https://github.com/Daniel-Krzyczkowski/WindowsIoT-AzureIoT/tree/master/WindowsIoTCore/FaceDetector)

I extended previous application (Motion Detector) with connected camera device and Microsoft Cognitive Services Face API. There is one more service added to the project called FaceRecognitionService (as in the Face Identifier app) which helps detect people from the images taken by camera device once motion is detected:

```csharp
public class FaceRecognitionService
    {
        public FaceRecognitionService()
        {
            FaceServiceClient = new FaceServiceClient("<<API key here>>", "https://westcentralus.api.cognitive.microsoft.com/face/v1.0");
        }

        public FaceServiceClient FaceServiceClient { get; }

        public async Task<string> VerifyFaceAgainstTraindedGroup(string personGroupId, Stream stream)
        {
            try
            {
                var faces = await FaceServiceClient.DetectAsync(stream);
                var faceIds = faces.Select(face => face.FaceId).ToArray();

                var results = await FaceServiceClient.IdentifyAsync(personGroupId, faceIds);
                foreach (var identifyResult in results)
                {
                    Console.WriteLine("Result of face: {0}", identifyResult.FaceId);
                    if (identifyResult.Candidates.Length == 0)
                    {
                        return "No one identified";
                    }
                    else
                    {
                        // Get top 1 among all candidates returned
                        var candidateId = identifyResult.Candidates[0].PersonId;
                        var person = await FaceServiceClient.GetPersonAsync(personGroupId, candidateId);
                        return "Identified as: " + person.Name;
                    }
                }
                return "No one identified";
            }

            catch(FaceAPIException ex)
            {
                return "An error has occurred: " + ex.Message;
            }
        }
    }
```

AzureIoTHubService class was extended with one more method - written to uplad taken image to the Azure Storage:

```
     public async Task SendImageToAzure(Stream imageStream)
     {
          await _deviceClient.UploadToBlobAsync($"Person.jpg", imageStream);
     }
```

In the MainPage class there is a method which prepares data (Face API analyzis result and image) to be send to the Azure IoT Hub:

```csharp
        private async Task SendImageWithAnalysis(string analyzisResult)
        {
            var personPicture = await GetImageStream();
            var motionEvent = new MotionEvent
            {
                RoomNumber = 1,
                ImageAnalyzisResult = analyzisResult
            };
            await _azureIoTHubService.SendImageToAzure(personPicture);
            await _azureIoTHubService.SendDataToAzure(motionEvent);
        }
```

### Microsoft Azure services configuration

In this section you will find which services were used in the Azure cloud and how to configure them. I assume that you have active Azure subscription.

### Azure IoT Hub

We will use Azure IoT Hub created previously - if you do not have one please refer to the [previous article](https://daniel-krzyczkowski.github.io/cloudyofthings/article1/index) where all required steps are described. We need to add file upload support (connect IoT Hub with Azure Storage Account) so please follow below steps:

In the messaging setion select File upload then select Azure Storage Container:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector1.PNG?raw=true" alt="Face detector"/>
</p>

Then if you do not have storage already select +Storage account and create it. If you already have one select it from the list:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector3.PNG?raw=true" alt="Face detector"/>
</p>

Click +Container and name it. This is the container where we will store taken image from IoT device:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector4.PNG?raw=true" alt="Face detector"/>
</p>

Select this container so IoT Hub will know where to store images:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector5.PNG?raw=true" alt="Face detector"/>
</p>

Save all settings:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector6.PNG?raw=true" alt="Face detector"/>
</p>

Azure IoT Hub is now connected with Azure Storage Account. Once image is taken by IoT device it is sent to the IoT Hub and then stored in the blob container.

### Azure Function IoTHubTrigger

Once there is new information sent to the Azure IoT Hub we want to trigger Function App. Here we will use IoTHubTrigger.

Open Functions blade and select "+" button to add new function:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetectorFuncApp.PNG?raw=true" alt="Face detector"/>
</p>

Select "Event Hub Trigger C#" template. Type the name and select IoT Hub for the Azure Event Hubs Trigger:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector0.PNG?raw=true" alt="Face detector"/>
</p>

That's it! Now you can paste the code responsible for sending Face API analyzis received from the device to the Azure Logic App:

```csharp
using System.Net;
using Microsoft.Extensions.Logging.Abstractions;
using System.Text;

private static HttpClient httpClient = new HttpClient();

public static void Run(string myEventHubMessage, ILogger logger)
{
    logger.LogInformation($"C# Event Hub trigger function processed a message: {myEventHubMessage}");

    HttpContent content = new StringContent(myEventHubMessage, Encoding.UTF8, "application/json");
    var response = httpClient.PostAsync("<<ADDRESS OF THE LOGIC APP TO HANDLE REQUESTS>>", content).Result; 
    if (response.IsSuccessStatusCode)
    {
      logger.LogInformation("Status from the Logic App: " + response.StatusCode);
    }

}

```
Remember to add project.json file with below content so NuGet packages are correctly added:

```csharp
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Newtonsoft.Json": "11.0.2",
        "Microsoft.Extensions.Logging.Abstractions": "2.1.1"
      }
    }
   }
}
```
We will store the logs in the Application Insights - if you do not have one please again refer to the [previous article.](https://daniel-krzyczkowski.github.io/cloudyofthings/article1/index)


### Azure Logic App

Now its time to configure Azure Logic App to send e-mail notifications with Face API analyzis result and taken image attached to the message. Go to the IoT Hub resource in Azure portal and open Events section and click Logic App tile:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector7.PNG?raw=true" alt="Face detector"/>
</p>

Fill required data including name of the Logic App:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector12.PNG?raw=true" alt="Face detector"/>
</p>

Then select Templates from the top:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector8.PNG?raw=true" alt="Face detector"/>
</p>

Select When a HTTP request is received trigger:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector9.PNG?raw=true" alt="Face detector"/>
</p>

This will generate first step for the Logic App:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector10.PNG?raw=true" alt="Face detector"/>
</p>

We need to define JSON schema for received messages:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector11.PNG?raw=true" alt="Face detector"/>
</p>

Paste below schema and save changes. Note that here we expecting to receive analyzis result and room number where motion was detected:

```csharp
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "properties": {
        "ImageAnalyzisResult": {
            "type": "string"
        },
        "RoomNumber": {
            "type": "integer"
        }
    },
    "required": [
        "RoomNumber",
        "ImageAnalyzisResult"
    ],
    "type": "object"
}
```

Add next step to the Logic App. We want to take image stored in the Azure Storage by name:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector13.PNG?raw=true" alt="Face detector"/>
</p>

Add next step called Send an email. Here you have to sign in to your Outlook account so you need to have one:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector14.PNG?raw=true" alt="Face detector"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector15.PNG?raw=true" alt="Face detector"/>
</p>

As you can see below there is body included with the analyzis result and room number where motion was detected.There is also image attached:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector16.PNG?raw=true" alt="Face detector"/>
</p>

Once email is sent we want to remove image from the Azure Storage so next step has to be added:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector18.PNG?raw=true" alt="Face detector"/>
</p>

Final steps of the Logic App should look like below:

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector19.PNG?raw=true" alt="Face detector"/>
</p>

## Final project

Once you scan and register yourself with Face Identifier UWP application, launch IoT device with Face Detector app. When motion is detected camera takes picture and analyzis is started. You should receive an email after short time:


<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetectorApp3.JPG?raw=true" alt="Face detector test"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetectorApp2.png?raw=true" alt="Face detector app"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article2/assets/FaceDetector20.PNG?raw=true" alt="Face detector app"/>
</p>


[Back](https://daniel-krzyczkowski.github.io/)
