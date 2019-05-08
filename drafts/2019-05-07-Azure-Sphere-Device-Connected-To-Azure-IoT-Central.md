---
title: "Azure Sphere Device Connected To Azure IoT Central"
---

<p align="center">
<img src="/images/cloudyofthings/article6/assets/IoTCentralWithAzureSphere1.PNG?raw=true" alt="Azure Sphere Device Connected To Azure IoT Central"/>
</p>

**TL;DR** 

Some time ago I jumped into Internet of Things topics connected with Microsoft Azure cloud. As a result I created my own series of articles about Azure IoT called **Cloudy of Things**.

You can read them all on this blog.

In this article you can find my latest project in which I connected Azure Sphere IoT device to the Azure IoT Central service to display data from sensors and take up some actions.

[Source code for this project is available here](https://github.com/Daniel-Krzyczkowski/WindowsIoT-AzureIoT/tree/master/AzureIoT/AzureSphereProject/SRC)

---

## Project structure and architecture

<p align="center">
<img src="/images/cloudyofthings/article6/assets/IoTCentralWithAzureSphere2.png?raw=true" alt="Solution architecture"/>
</p>

Above diagram presents the whole solution. As you can see it is simple - Azure Sphere IoT device is directly connected to the Azure IoT Central. This is huge advantage because we do not have to create complex architecture.

### Microsoft Azure Sphere

In this project I used Microsoft Azure Sphere Development Board from [Seeed Studio](https://www.seeedstudio.com/Azure-Sphere-MT3620-Development-Kit-EU-Version-p-3134.html)

Temperature/Humidity sensor and OLED screen were took from [Grove Starter Kit for Azure Sphere MT3620 Development Kit](https://www.seeedstudio.com/Grove-Starter-Kit-for-Azure-Sphere-MT3620-Development-Kit-p-3150.html)

### Azure IoT Central connection with Azure Sphere device

There are few prerequisites you have to complete to be able to display data from sensors connected to the Azure Sphere in the Azure IoT Central portal:

1. Create Azure IoT Central application
2. Download the tenant authentication CA certificate
3. Upload the tenant CA certificate to Azure IoT Central and generate a verification code
4. Verify the tenant CA certificate
5. Use the validation certificate to verify the tenant identity

Once steps above are completed you can configure sample application from my repository to work with your Azure Sphere device.
All above steps are well documented by Microsoft under [this](https://docs.microsoft.com/en-us/azure-sphere/app-development/setup-iot-central#step-4-verify-the-tenant-ca-certificate) link.

Once Azure IoT Central application is ready and you versified Azure Sphere tenant with certificate we can proceed with code sample setup.

To configure the sample application, you'll need to supply the following information in the app_manifest.json file for AzureIoT:

The Tenant ID for your Azure Sphere device
The Scope ID for your Azure IoT Central application
The IoT hub URL for your Azure IoT Central application
The Azure DPS global endpoint address
Follow these steps to gather the information and configure the application:

1. Open AzureIoT.sln in Visual Studio.

2. In Solution Explorer, find the app_manifest.json file and open it.

3. In an Azure Sphere Developer Command Prompt, issue the following command to get the tenant ID. Copy the returned value and paste it into the DeviceAuthentication field of the app_manifest.json file:

4. azsphere tenant show-selected

5. In an Azure Sphere Developer Command Prompt, run the ShowIoTCentralConfig.exe program from the sample repository (the ShowIoTCentralConfig program is located in the AzureIoT\Tools folder). Just type: ShowIoTCentralConfig

When prompted, log in with the credentials you use for Azure IoT Central.

Copy the information from the output into the app_manifest.json file in Visual Studio.
In the manifest file you should see that there is new address added to the "AllowedConnections" section that targets your Azure IoT Central application.

### Prepare Azure IoT Central dashboard to display data from humidity and temperature sensors

Once device is connected with the Azure IoT Central we can prepare dashboards to display temperature and humidity values. Below steps present how to do it:

1. Open "Device templates" tab:

2. Click "New Measurement"

3. Select "Telemetry"

4. Provide details about temperature telemetry:

Now repeat the same above steps for the humidity:


