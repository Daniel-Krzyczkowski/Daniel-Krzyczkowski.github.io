# Industry 4.0 - Connected Factory with Microsoft IoT Solutions


![Image](https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/mainassets/CloudyOfThings.png?raw=true)

## Introduction

Before I introduce the Microsoft Azure cloud services for Internet of Things (IoT), I would like to start from Industry 4.0 term.

#### Industry 4.0

Industry 4.0 is a term connected with the Industrial Revolution. Please lets first look at below image. There are four industrial revolutions presented. In this article we are talking about fourth one.

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article4/assets/Industry1.PNG?raw=true" alt="Industry revolution table"/>
</p>

The fourth industrial revolution - term which refers to the concept of the "industrial revolution" in connection with the contemporary mutual use of automation, data processing and exchange as well as manufacturing techniques. Definitionally, it is a collective term for the techniques and principles of the functioning of a value chain organization using or using cyber-physical systems, the Internet of Things (IoT) and cloud computing.
Openness and interoperability between hardware, software, and services will be key in helping manufacturers transform how they operate and create solutions that benefit productivity. It is crucial to provide unified way to communicate between hardware, software and what is more - to communicate with the cloud services.

## Open Platform Configurations Unified Architecture - OPC UA

#### OPC Foundation

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article4/assets/Industry3.PNG?raw=true" alt="OPC Foundation"/>
</p>

OPC Foundation - an organization whose task is to provide broad opportunities for interoperability in the field of automation, by creating and maintaining an open communication standard that allows transfer of process data, alarm and event data, historical data and batch data to devices from different manufacturers.

[OPC Foundation official website](https://opcfoundation.org/)

#### Open Platform Configurations Unified Architecture (OPC UA)

The OPC UA standard is driven by the OPC Foundation described above.
Adapting machines to become compatible with OPC UA is an un-intrusive and cost-effective way to connect factory assets and adapters to field buses are available through a rich ecosystem of vendors (mentioned below in the article). OPC UA is also called communication technology for Industry 4.0 and is essential to reaching this next level of connectivity in manufacturing facilities.

<p align="center">
  <img src="https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article4/assets/Industry4.PNG?raw=true" alt="OPC Foundation"/>
</p>

There is a great video created by OPC Foundation which presents the whole idea of OPC UA (Open Platform Configurations Unified Architecture):

[![OPC UA](https://github.com/Daniel-Krzyczkowski/Daniel-Krzyczkowski.github.io/blob/master/cloudyofthings/article4/assets/Industry2.jpg)](https://www.youtube.com/watch?v=-tDGzwsBokY "OPC UA")

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

## OPC UDA with Microsoft Azure cloud

I mentioned that Industry 4.0 is about integration and communication betweend hardware, software and the cloud. This is why Microsoft announced to invest huge amount of money (5 billion $) in the IoT solutions. 

With reference to the Azure cloud platform, it offers dedicated services for IoT. Below I described some of them. It is worth to mentioned that all these services were build with commitment to the OPC UA standards.

#### Azure IoT Hub

he Azure IoT Hub provides reliable and secure communication between IoT devices. It also establishes bi-directional communication between each device and the Azure cloud. With Azure IoT Hub you can send messages:

- from a device to the cloud – e.g. temperature values provided by a sensor connected to an IoT device, sent for analysis, and
- from the cloud to a device – e.g. a message with software update payload.

#### Azure IoT Edge

Azure IoT Edge enables moving cloud analytics and custom business logic to IoT devices. The device can process logic directly without pushing data to the cloud.

## OPC UDA Standard Devices

Aaa

## Microsoft Azure IoT Suite Connected Factory

Aaa

## Microsoft Azure IoT protocol gateway

Aaa

## Additional resources

Aaa


[Back](https://daniel-krzyczkowski.github.io/cloudyofthings/main/index)
