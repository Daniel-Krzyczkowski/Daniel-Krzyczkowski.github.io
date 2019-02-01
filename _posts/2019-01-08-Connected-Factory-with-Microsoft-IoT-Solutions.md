---
title: "Industry 4.0 - Connected Factory with Microsoft IoT Solutions"
---

![Image](/images/cloudyofthings/article4/assets/CloudyOfThingsArticle4.png?raw=true)

## Introduction

Before I introduce the Microsoft Azure cloud services for Internet of Things (IoT), I would like to start from Industry 4.0 term.

#### Industry 4.0

Industry 4.0 is a term connected with the Industrial Revolution. Please lets first look at below image. There are four industrial revolutions presented. In this article we are talking about fourth one.

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry1.PNG?raw=true" alt="Industry revolution table"/>
</p>

The fourth industrial revolution - term which refers to the concept of the "industrial revolution" in connection with the contemporary mutual use of automation, data processing and exchange as well as manufacturing techniques. Definitionally, it is a collective term for the techniques and principles of the functioning of a value chain organization using or using cyber-physical systems, the Internet of Things (IoT) and cloud computing.
Openness and interoperability between hardware, software, and services will be key in helping manufacturers transform how they operate and create solutions that benefit productivity. It is crucial to provide unified way to communicate between hardware, software and what is more - to communicate with the cloud services.

## Open Platform Configurations Unified Architecture - OPC UA

#### OPC Foundation

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry3.PNG?raw=true" alt="OPC Foundation"/>
</p>

OPC Foundation - an organization whose task is to provide broad opportunities for interoperability in the field of automation, by creating and maintaining an open communication standard that allows transfer of process data, alarm and event data, historical data and batch data to devices from different manufacturers.

[OPC Foundation official website](https://opcfoundation.org/)

#### Open Platform Configurations Unified Architecture (OPC UA)

The OPC UA standard is driven by the OPC Foundation described above.
Adapting machines to become compatible with OPC UA is an un-intrusive and cost-effective way to connect factory assets and adapters to field buses are available through a rich ecosystem of vendors (mentioned below in the article). OPC UA is also called communication technology for Industry 4.0 and is essential to reaching this next level of connectivity in manufacturing facilities.

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry9.PNG?raw=true" alt="OPC Foundation"/>
</p>

There is a great video created by OPC Foundation which presents the whole idea of OPC UA (Open Platform Configurations Unified Architecture):

[![OPC UA](/images/cloudyofthings/article4/assets/Industry2.jpg?raw=true)](https://www.youtube.com/watch?v=-tDGzwsBokY "OPC UA")

One more important aspect - security. It is confirmed that OPC UA is secure by default. It means that all security features are turned on and already configured so there is no need to do this step manually and sees how an end-to-end solution can be secured. It was also confirmed by German Federal Office for Information Security (BSI):

*"An extensive analysis of the security functions in the specification of OPC UA confirmed that OPC UA was designed with a focus on security and does not contain systematic security vulnerabilities."*

You can read more about it [here](https://opcconnect.opcfoundation.org/2016/06/bsi-security-check/).

## Open-source cross-platform OPC UA support from Microsoft

Microsoft has a long-standing partnership with the OPC Foundation is currently the number one open-source contributor to the OPC Foundation. Microsoft is also the only cloud vendor that uses both OPC UA client-server connections as well as the new OPC UA publish-subscribe connections all the way to the cloud and back. It means that Microsoft is in a unique position to make the rich OPC UA information model - for instance the semantic description of the machines, available in the cloud for enabling services (like Azure IoT Hub).

At Hannover Messe 2016 Microsoft worked closely with the OPC Foundation to expand product support of the OPC UA open-source software stack, which enables deep integration with Azure IoT, as well as the Universal Windows Platform (UWP). Microsoft will contribute a .NET Standard reference stack to the OPC Foundation GitHub open-source. As .NET Standard is platform-independent, this stack has the benefit of working on all common software platforms in market today, allowing the creation of OPC UA clients and servers.

## List of Open Source OPC UA Implementations

There are many open source implementations of OPC UA (not only provided by Microsoft) in the different languages and with different licenses:
#### open62541 - C library

[open62541 - C library](http://open62541.org/)

#### UA.NET Standard - C# library (.NET Standard)

[UA.NET Standard - C# library (.NET Standard)](https://github.com/OPCFoundation/UA-.NETStandard)

#### node-opcua - JavaScript

[node-opcua - JavaScript](http://node-opcua.github.io/)

#### FreeOpcUa - C++ library

[FreeOpcUa - C++ library](http://freeopcua.github.io/)

#### Python FreeOpcUa - Python library

[Python FreeOpcUa - Python library](https://github.com/FreeOpcUa/python-opcua)

#### Eclipse Milo - Java

[Eclipse Milo - Java](https://github.com/eclipse/milo)

Full list can be found [here](https://github.com/open62541/open62541/wiki/List-of-Open-Source-OPC-UA-Implementations).

## OPC UA with Microsoft Azure cloud

I mentioned that Industry 4.0 is about integration and communication betweend hardware, software and the cloud. This is why Microsoft announced to invest huge amount of money (5 billion $) in the IoT solutions. 

With reference to the Azure cloud platform, it offers dedicated services for IoT. Below I described some of them. It is worth to mentioned that all these services were build with commitment to the OPC UA standards.

### Azure IoT Hub

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry7.png?raw=true" alt="Azure IoT Hub"/>
</p>

The Azure IoT Hub provides reliable and secure communication between IoT devices. It also establishes bi-directional communication between each device and the Azure cloud. With Azure IoT Hub you can send messages:

- **from a device to the cloud** – e.g. temperature values provided by a sensor connected to an IoT device, sent for analysis, and
- **from the cloud to a device** – e.g. a message with software update payload.

### Azure IoT Edge

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry6.png?raw=true" alt="Azure IoT Edge"/>
</p>

Azure IoT Edge enables moving cloud analytics and custom business logic to IoT devices. The device can process logic directly without pushing data to the cloud.

**The Azure IoT Edge runtime**

The runtime enables custom logic and cloud logic on IoT Edge devices. It is located on the IoT Edge device, and executes management and communication operations. This can include maintaining Azure IoT Edge security standards on the device, installing and updating workloads or facilitating communication between the device and the cloud.

**Module**
IoT Edge modules are units that consist of custom logic (for instance to analyze temperature) or cloud logic (like Azure Functions, Azure Stream Analytics and Azure Machine Learning). The Azure Container Registry stores these modules as Docker containers.
When a module is being deployed on a device, the IoT Hub contacts the Azure IoT Edge runtime, which in turn pulls the image from the Azure Container Registry and starts running it.

**IoT Hub**

The Azure IoT Edge runtime connects to Azure IoT Hub to facilitate communication between the Edge device and the cloud. If data has to be pushed to the cloud or a new module needs to be deployed on the device, it is done through the IoT Hub.

## OPC UA Standard Devices

Microsoft works close with IoT devices manufacturers to provide easier way to deploy OPC UA standards and to easier integration with Azure cloud services. These all devices are compatible with OPC UA standards and ready for production usage. They can be used as a proxy between factory fields and the Azure cloud.

[Full list of OPC compatible devices](https://catalog.azureiotsolutions.com/alldevices?q=opc)

### Industrial IoT Starter Kit

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry13.jpg?raw=true" alt="IoT Starter Kit 2"/>
</p>

To make it easier to start integration and to verify your specific requirements there is production-ready starter kit created in cooperation between Microsoft, Softing and Hewlett Packard. It enables connecting your existing IoT devices with Azure cloud - it is working as a proxy between devices and the cloud. Of course there are other devices like that too - ready for production use.

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry8.png?raw=true" alt="IoT Starter Kit 2"/>
</p>

As you can see in the picture above there are two important modules in this architecture:

**OPC Proxy Module**

OPC Proxy Module enables bi-directional communication between Azure cloud (Azure IoT Hub in this case) and IoT machines and devices. It means that with Proxy module we are able to collect data from the devices and send some data to them form the Azure cloud.

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry10.PNG?raw=true" alt="OPC Proxy Module"/>
</p>

**OPC Publisher Module**

OPC Publisher Module integrates with existing IoT machines and devices, collects the data generated by them and then converts this data so it can be send to the Azure cloud.

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry12.PNG?raw=true" alt="OPC Publisher Module"/>
</p>

It is important to mention that OPC has been based on a client/server architecture, and in the 2016 its architecture was enhanced with the inclusion of publish/subscribe to provide a solid infrastructure. This allows information integration from embedded devices to the cloud.

**Industrial IoT Starter Kit device works as both: OPC Proxy and OPC Publisher.**

[Read full IoT Industrial specification](https://data-intelligence.softing.com/products/iot-gateways/industrial-iot-starter-kit/#tx-dftabs-tabContent1)

I also recommend to see: [Industrial Interoperability from Sensor to Cloud
OPC UA Integration into Azure IoT Suite document](https://opcfoundation.org/wp-content/uploads/2016/10/Microsoft-OPC-UA-5-Clicks-To-Digital-Factory.pdf)

### OPC UA Gateway Partners

You can see that UPC UA standards are adopted by increasing number of companies:
<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry11.PNG?raw=true" alt="OPC UA Gateway Partners"/>
</p>

## Microsoft Azure IoT Suite Connected Factory

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry14.PNG?raw=true" alt="Microsoft Azure IoT Suite Connected Factory"/>
</p>

Connected Factory is one of the IoT Solution Accelerators created and provided by Microsoft. The IoT solution accelerators are complete, ready-to-deploy IoT solutions that implement common IoT scenario. 

Currently, there are four solution accelerators available for you to deploy:

- [Remote Monitoring](https://docs.microsoft.com/en-us/azure/iot-accelerators/about-iot-accelerators#remote-monitoring)

- [Connected Factory](https://docs.microsoft.com/en-us/azure/iot-accelerators/about-iot-accelerators#connected-factory)

- [Predictive Maintenance](https://docs.microsoft.com/en-us/azure/iot-accelerators/about-iot-accelerators#predictive-maintenance)

- [Device Simulation](https://docs.microsoft.com/en-us/azure/iot-accelerators/about-iot-accelerators#device-simulation)

In this article we will concentrate on the Connected Factory.

### Logical architecture

Below there is presented logical architecture of the Connected Factory. Please note that in this case Proxy module is used to communicate with siulated devices (there is bi-directional communication):

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry15.png?raw=true" alt="Logical architecture"/>
</p>

To collect telemetry data from the devices Publisher module is used (one way communication from the devices to the cloud):

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry16.png?raw=true" alt="Proxy architecture"/>
</p>

I recommend to check great Microsoft documentation available [here.](https://docs.microsoft.com/en-us/azure/iot-accelerators/iot-accelerators-connected-factory-sample-walkthrough)

#### Security aspect

It is crucial to mention about additional security aspects here. As I mentioned before OPC UA is secure by default. Microsoft additionally made whole Connected Factory solution secured by default - it means that when adopting this solutuion customers do not have to worry about security. What is more Microsoft Security Development Lifecycle (SDL) is applied so many different security aspects are tested and verified constantly. Additional security services such like Microsoft Azure Security & Trust Center.

I recommend to watch the video called Azure Industrial IoT in details by Erich Barnstedt from Microsoft:

[![I recommend to watch the video called Azure Industrial IoT in details by Erich Barnstedt, Microsoft](/images/cloudyofthings/article4/assets/Industry18.PNG?raw=true)](https://www.youtube.com/watch?v=QJ1DWTvGQxo "I recommend to watch the video called Azure Industrial IoT in details by Erich Barnstedt, Microsoft")

[You should also visit Microsoft Security Development Lifecycle website](https://www.microsoft.com/en-us/securityengineering/sdl/practices)


## Microsoft Azure IoT protocol Gateway

There are some cases in which devices or field gateways might not be able to use one of these standard protocols and require protocol adaptation. Azure IoT Hub natively supports communication over the MQTT, AMQP, and HTTPS protocols and in case where this protocols cannot be used directly Microsoft Azure IoT Protocol Gateway is needed. The Azure IoT protocol gateway provides a programming model for building custom protocol adapters for variety of protocols.

More information about how Protocol Gateway works can be found [here.](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)

<p align="center">
  <img src="/images/cloudyofthings/article4/assets/Industry17.jpg?raw=true" alt="Logical architecture"/>
</p>


## Additional resources and summary

Below there are some interesting resources which are worth to check:

[IoT Platforms in Europe (preliminary results)](https://www.pac-online.com/sites/pac-online.com/files/upload_path/PDFs/Status%20Quo%20of%20IoT%20Platforms%20Webinar%20170706.pdf)

[Industrial Interoperability from Sensor to Cloud
OPC UA Integration into Azure IoT Suite](https://opcfoundation.org/wp-content/uploads/2016/10/Microsoft-OPC-UA-5-Clicks-To-Digital-Factory.pdf)

[Secured IoT Interoperability From Sensor to IT Enterprise](https://www.industryofthingsworld.com/wp-content/uploads/2018/10/Day-02_Stream-1_Stefan-Hoppe_OPC-Foundation.pdf)

[Choosing Your Messaging Protocol: AMQP, MQTT, or STOMP](https://blogs.vmware.com/vfabric/2013/02/choosing-your-messaging-protocol-amqp-mqtt-or-stomp.html)

I tried to include all helpful information in this article from many different sources but of course this is just a drop in the ocean. There is a lot more to learn and investigate to properly integrate cloud services with factories and IoT Enterprise solutions.


[Back](https://daniel-krzyczkowski.github.io/)
