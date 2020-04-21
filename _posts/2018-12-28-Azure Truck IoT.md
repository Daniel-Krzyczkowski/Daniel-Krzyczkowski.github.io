---
title: "Azure Truck IoT"
excerpt: "Azure Truck project was created to demonstrate how Microsoft technologies can be used together inside the car created by polish Microsoft Most Valuable Professionals."
header:
  image: /images/cloudyofthings/article3/assets/CloudyOfThingsArticle3.png
---

![Image](/images/cloudyofthings/article3/assets/CloudyOfThingsArticle3.png?raw=true)

![Image](/images/cloudyofthings/article3/assets/AzureTruckIoT1.jpg?raw=true)

## Project description

Azure Truck project was created to demonstrate how Microsoft technologies can be used together inside the car created by polish Microsoft Most Valuable Professionals. This part of the whole project describes Internet of Things (IoT) part which was created especially for Azure Truck.


## Solution

<p align="center">
  <img src="/images/cloudyofthings/article3/assets/AzureTruckIoT6.png?raw=true" alt="Solution diagram"/>
</p>

IoT solution for Azure Truck consists of three Raspberry Pi devices connected with the Microsoft Azure Cloud. Each board has some sensors connected:

#### First board has below components:

- HDMI Camera 

- LCD screen 

- Motion detector 

#### Second board has below components:

- RGB color LED 

- Motion detector 

- Temperature and pressure sensors

- Color detector

#### Third board has below components:

- Temperature sensors

All three devices are connected to the Azure cloud services but third one is configured as "Edge" device.


## Microsoft Azure cloud services for IoT

As you can see on the architecture diagram each device is connected with Azure cloud. Below I described each Azure service used in Azure Truck IoT solution.

#### Azure IoT Hub

The Azure IoT Hub provides reliable and secure communication between IoT devices. It also establishes bi-directional communication between each device and the Azure cloud.

#### Azure Storage

Azure Storage was created to store data collected from the sensors (like temperature or pressure). This type of data is stored in the Azure Table Storage. Azure Blob Storage was created to keep images send from the IoT device with motion sensor and a camera.

#### Azure Function for face detection

First Azure Function App was created to use Azure Cognitive Services Face API to detect person from the image uploaded to the Azure Blob Storage. Once there is a new photo uploaded to the Storage, Function App is triggered. Face API is called and result about face detection is retuned through Azure IoT Hub to the IoT device with camera and LCD screen.

#### Azure Function for temperature, pressure, altitude and color data collection

Second Azure Function App was created to collect data from the device where pressure, temperature, altitude and color sensors are connected. Once there is new information sent from the device, Function App is invoked and data is stored in the Azure Table Storage. This is also the main data source for Power BI dashboards.

#### Azure IoT Edge

As mentioned before one of the devices was used as an "Edge" device. In this case we used Azure IoT Edge. Azure IoT Edge enables moving cloud analytics and custom business logic to IoT devices. The device can process logic directly without pushing data to the cloud.
Azure IoT Edge consists of three main components:

- Azure IoT Edge runtime

The runtime enables custom logic and cloud logic on IoT Edge devices. It is located on the IoT Edge device, and executes management and communication operations.

- Azure IoT Edge module (or modules)

IoT Edge modules are units that consist of custom logic (for instance to analyze temperature) or cloud logic (like Azure Functions, Azure Stream Analytics and Azure Machine Learning). In our case there was temperature module writted in Python to detect temperature for the sensor.

- IoT Hub

The Azure IoT Edge runtime connects to Azure IoT Hub to facilitate communication between the Edge device and the cloud.

- Azure Container Registry

Modules are kept as a Docker images in the Azure Container Registry. Azure IoT Edge Runtime can pull images from here and run modules as Docker container.

<p align="center">
  <img src="/images/cloudyofthings/article3/assets/AzureTruckIoT5.png?raw=true" alt="Solution diagram"/>
</p>


## Devices specification

There was two Raspberry Pi2 devices and one Raspberry Pi3 device used in the Azure Truck project. Two of them running Windows 10 IoT Core system and on of them ("Edge device") is running Raspian9. For the first two devices there is dedicated Universal Windows Application (UWP) created with IoT extension to provide communication between the app and the device and sensors. "Edge" device with IoT Edge runtime has module written in Python.

Below there is a list of IoT sensors used in this project together with connection schemas.

<p align="center">
  <img src="/images/cloudyofthings/article3/assets/AzureTruckIoT3.JPG?raw=true" alt="Solution diagram"/>
</p>

#### Face detection devices

- HD USB Camera
- [Motion sensor](https://learn.adafruit.com/pir-passive-infrared-proximity-motion-sensor/overview)
- [LCD screen](https://www.adafruit.com/product/181)

<p align="center">
  <img src="/images/cloudyofthings/article3/assets/FaceDetectorDeviceSchema.png?raw=true" alt="FaceDetectorDeviceSchema"/>
</p>

#### Sensors device (temperature, pressure, altitude and color)

- [Temperature and pressure sensor](https://learn.adafruit.com/adafruit-bmp280-barometric-pressure-plus-temperature-sensor-breakout/overview)

- [RGB color detection sensor](https://www.adafruit.com/product/1334)

- RGB diode

<p align="center">
  <img src="/images/cloudyofthings/article3/assets/SensorsDeviceSchema.png?raw=true" alt="SensorsDeviceSchema"/>
</p>

#### Temperature Edge IoT device

- [Temperature sensor](https://www.adafruit.com/product/165)

<p align="center">
  <img src="/images/cloudyofthings/article3/assets/IoTEdgeDeviceSchema.png?raw=true" alt="IoTEdgeDeviceSchema"/>
</p>

## Project source code

- Azure Truck IoT UWP application source code can be found in our official Github repository [here](https://github.com/Daniel-Krzyczkowski/WindowsIoTCore/tree/master/AzureTruckIoT)

- Python module source code for the "Edge" device can be found in in our official Github repository [here](https://github.com/AzureTruck/IoT/tree/master/TemperatureEdgeSolution)

## Final project

All devices were packed and mounted in our Azure Truck. You can see the final result below.

<p align="center">
  <img src="/images/cloudyofthings/article3/assets/AzureTruckIoT7.png?raw=true" alt="Solution diagram"/>
</p>

<p align="center">
  <img src="/images/cloudyofthings/article3/assets/AzureTruckIoT8.png?raw=true" alt="Solution diagram"/>
</p>

I recommend to check our official Azure Truck Github repository, where presentations and samples are published and maintained:
[Azure Truck Github](https://github.com/AzureTruck)

[Back](https://daniel-krzyczkowski.github.io/)
