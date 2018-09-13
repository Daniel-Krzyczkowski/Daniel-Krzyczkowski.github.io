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

## Code and Configuration
In this section you will find how to create solution described above.

###Motion Detector UWP application for Windows IoT Core
To detect motion from the sensor connected to the Raspberry Pi I wrote a Universal Windows Platform application. Whole source code you can find on my GitHub here.

## Demo
Aaa

## Summary
Aaa

[Back](https://daniel-krzyczkowski.github.io/cloudyofthings/main/index)
