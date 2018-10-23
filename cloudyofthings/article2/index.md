# Face detector - automatic e-mail alerts with Windows IoT Core and Microsoft Azure cloud


![Image](https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/mainassets/CloudyOfThings.png?raw=true)

## Use case

In my previous article (you can find it [here](https://daniel-krzyczkowski.github.io/cloudyofthings/article1/index)) I described how to connect motion sensor to the device with Windows IoT Core system and how to connect it with Microsoft Azure IoT Hub to send SMS alerts once motion is detected.

In this article I would like to extend previous sample with face detection using camera connected to the IoT device and Microsoft Cognitive Serivces (Face API service). Once motion is detected, camera takes picture which is analyzed with Cognitive Services Face API. At the end there is an email notification send with taken image and analyzis result using Azure Logic App.

## Solution

In the previous article I presented how to connect motion sensor with Raspberry Pi 2 device running Windows IoT Core system. This time we will add one more element - camera device. Once motion is detected by sensor - photo is taken and analyzed with Microsoft Cognotive Services Face API. Then image together with analyzis result is sent to the Azure IoT Hub and then used with other components.

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
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article2/assets/FaceDetectorApp1.png?raw=true" alt="Face detector app"/>
</p>

As you can see I scaned my face and registered myself as "Daniel" in the "Home" group.

### Face Detector UWP application for Windows IoT Core

Aaa

### Microsoft Azure services configuration

In this section you will find which services were used in the Azure cloud and how to configure them. I assume that you have active Azure subscription.

### Azure IoT Hub

Aaa

### Azure Function IoTHubTrigger

Aaa

### Azure Logic App

Aaa

## Final project

Aaa


[Back](https://daniel-krzyczkowski.github.io/cloudyofthings/main/index)
