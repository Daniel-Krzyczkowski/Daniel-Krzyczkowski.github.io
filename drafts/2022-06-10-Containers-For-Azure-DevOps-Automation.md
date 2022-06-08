---
title: "Azure Hints Series - Containers for Azure DevOps Automation"
excerpt: "This article presents how to setup Azure API Management with custom domain behind Azure Front door"
header:
  image: /images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation.png
---

<p align="center">
<img src="/images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation.png?raw=true" alt="Azure Hints Series - Containers for Azure DevOps Automation"/>
</p>

# Introduction

Probably for most of us containers are associated with building cloud native applications and microservices platforms. The power of containers portability makes them attractive when building and architecting cloud solutions. However they can be really helpful in other scenarios too, like DevOps automation. In this article I would like to talk about how containers together with Azure DevOps can help with DevOps automation when it comes to Azure cloud solutions.

# Why containers for DevOps automation?

First there can be a question why containers for DevOps automation? To answer this questions, let me set the stage a little bit.
Let's assume that we use Azure DevOps to keep our solution's source code together with Azure infrastructure code. Our solution consists of many different Azure Services operating withing Azure Virtual Network. To visualize this, here is simple diagram:

<p align="center">
<img src="/images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation-02.png?raw=true" alt="Image not found"/>
</p>

To keep our environment secure, Azure Virtual Network is used and Function Apps are integrated with VNET using Private Links. Probably you know this but with such architecture and configuration using Microsoft-Hosed Azure DevOps Agents can be problematic. Why? Because by default they will not be able to reach Azure Functions to deploy your code. Because of the fact that Azure resources are located in the Azure Virtual Network, direct access to them is blocked.

In this case we have three options to still be able to deploy our code using Azure DevOps pipelines:

1. Use [Azure DevOps service tag](https://devblogs.microsoft.com/devops/azure-devops-service-tag-released/) to enable access to Virtual Network from Azure DevOps. This is not perfect solution because we cannot enable access for specific Microsoft-Hosted Agent, we have to add whole range of IP addresses. You can read about it [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#networking). Sometimes this can be not acceptible due to security restrictions because of the fact that in this scenario our Virtual Network is opened for the whole region of Azure DevOps Microsoft-Hosted Agents, not owned by us.
 
2. Use Azure Virtual Machnine integrated with our VNET and install Self-Hosted Azure DevOps Agent. This will solve the problem with network access but we will face challenges related to VM cost, and maintenance.

3. Use Self-Hosted Agents as running containers. In this case we have three options to run them: with Azure Container Instances, Azure Container Apps, and with Azure Kubernetes Service. We will talk about each approach later in the article. For sure the benefit of such approach is reduced cost if we compare it to Virutal Machnines.


# Virtual Machnines vs Container services in Azure

Let's focus on two approaches when it comes to running Self-Hosted Agents in the Azure cloud to deploy our code in resources withing Azure Virtual Network.

## Virtual Machines for DevOps automation

The most popular way to use Azure DevOps Self-Hosted Agents is to host them using Azure Virtual Machine that is integrated with Azure Vritual Network. Because of this fact, we can easily deploy code to resources like Azure Web Apps, Function Apps in the same Azure Virtual Network.

Unfortunately, there are two significant problems with this aproach:

1. Cost - Azure Virtual Machine is costly. Here is the Linux machine with low configuration. As you can see, estimated monthly cost is around $13.19. Of course it can be lower/higher depending on usage but still if we assume that this will be $13.19 per month, per year the cost will be $158.28. If you decide to use Windows machine, the cost will increase.

<p align="center">
<img src="/images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation-03.PNG?raw=true" alt="Image not found"/>
</p>

2. Maintenance - as you probably know, Azure Virtual Machnies are under IaaS (Infrastructure as a Service) cloud service model. It means that there are many things we have to take care of like operating system updates, installation of proper applications, and security:

<p align="center">
<img src="/images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation-04.png?raw=true" alt="Image not found"/>
</p>

The positive side of this approach is the fact that we have full control over VM and tools that can be installed. In some scenarios this can be helpful when we have complex CI/CD process.


## Containers for DevOps automation

Event if containers are associated with building cloud native applications, they are also perfect match for running Self-Hosted Azure DevOps Agents. Azure cloud offers many container services but let's talk about three most popular and useful when it comes to DevOps automation.


### Azure Container Instances

<p align="center">
<img src="/images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation-05.png?raw=true" alt="Image not found"/>
</p>

Azure Container Instances let you run a container in Azure without managing virtual machines and without a higher-level service. Azure Container Instances is useful for scenarios that can operate in isolated containers, including simple applications, task automation, and build jobs used in Azure DevOps pipelines.

If we take a look at pricing, we can notice that cost is quite low if we compare it to Azure Virtual Machnine cost:

<p align="center">
<img src="/images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation-08.PNG?raw=true" alt="Image not found"/>
</p>

It is worth to know that when we create Azure Container Instances, we can choose to run one container or group of containers:

<p align="center">
<img src="/images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation-09.PNG?raw=true" alt="Image not found"/>
</p>

A container group is a collection of containers that get scheduled on the same host machine.
The containers in a container group share a lifecycle, resources, local network, and storage volumes. It means that we cannot scale them dynamically. Once Container Instance is created, there is no option to scale the number of groups (pods in Kubernetes world).

However to make it easier to understand the pricing, let's do the simple calculation provided by Microsoft calculator.

We create a Linux container group with a 1.3 vCPU, 2.15 GB configuration 50 times daily during a month (30 days). The container group duration is 150 seconds. In this example, the vCPU and memory usage must be rounded up to calculate total cost.

**Memory duration:**
Number of container groups * memory duration (seconds) * GB * price per GB-s * number of days

50 container groups * 150 seconds * 2.2 GB * $0.00000124 per GB-s * 30 days = $0.612

**vCPU duration:**
Number of container groups * vCPU duration (seconds) * vCPU(s) * price per vCPU-s * number of days

50 container groups * 150 seconds * 2 vCPU * $0.00001125 per vCPU-s * 30 days = $5.063

**Total billing:**
Memory duration (seconds) + vCPU duration (seconds) = total cost

**$0.612 + $5.063 = $5.675**

We can use Azure Container Instances to run Azure DevOps Self-Hosted Agents. In this case, once the Container Instance is created, we can schedule Azure DevOps Pipeline run. In the typical scenario we will run one Self-Hosted Agent in one Azure Container Instance:


<p align="center">
<img src="/images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation-10.PNG?raw=true" alt="Image not found"/>
</p>

It means that if we want more Self-Hosted Agents to handle scheduled Pipelines runs, we will have to create more Azure Container Instances. There is not dynamic/event driven scaling available. If we will have three separate jobs scheduled in our Pipelines in Azure DevOps, each job will be queded and will run after another.

It is not possible to scale a specific ACI instance. If you want more CPU/Memory you would need to redeploy that container again - this is one of the downsides.
There is also one and the biggest disadvantage of this Azure service when it comes to Azure DevOps Sel-Hosted Agents.

** There is no option to scale Azure Container Instances horizontally basing on the amount of pipeline runs pending in a given agent pool**. It means that we have to architect our pipelines in the way that they first create Azure Container Instance and execute actual job.

There are also some benefits over using IaaS VMs:

* It does not use public IP’s. It doesn’t need one, as the Azure DevOps agent initiates the communication to the service.
* It does not have any exposed ports. There’s no need for publishing anything.
* Can be provisioned very quickly: to fully configure a container instance with the required components takes 5-10 minutes.


To summarize, Azure Container Instances is perfect match when we need to run Azure DevOps Self-Hosted Agents, we do not want to maintain Azure Virutal Machnines, and we want to reduce the cost. It is worth to mention that [Azure Container Instances can be deployed to Azure Virtual Network](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-vnet) so we will be able to reach out to other resources in this network.


### Azure Container Apps

Personally, this is my favorite container service in the Azure cloud. Azure Container Apps enables you to run microservices and containerized applications on a serverless platform so you can forget about managing complex Kubernetes clusters but still benefit from Kubernetes concepts.

One of the biggest advantages is auto-scaling. Applications built on Azure Container Apps can dynamically scale based on the following characteristics:

* HTTP traffic
* Event-driven processing
* CPU or memory load
* Any KEDA-supported scaler

Azure Container Apps manages automatic horizontal scaling through a set of declarative scaling rules. As a container app scales out, new instances of the container app are created on-demand. These instances are known as replicas. When you first create a container app, the scale rule is set to zero. No charges are incurred when an application scales to zero.

Can we use Azure Container Apps to run Azure DevOps Self-Hosted Agents? Of course we can! Let's dive into the topic and see why Azure Container Apps service can be more beneficial than using Azure Container Apps.

**Scaling**

The biggest advantage over Azure Container Instances is the fact that we can automatically scale the number of containers with our Self-Hosted Agents. Here is a great place to mention that Azure Container Apps supports KEDA ScaledObjects and all of the available KEDA scalers. For those of you who are not familiar with KEDA, [here is a link](https://keda.sh/). KEDA is a Kubernetes-based Event Driven Autoscaler. With KEDA, you can drive the scaling of any container in Kubernetes based on the number of events needing to be processed.

In the Azure Container Apps documentation we can find section with explanation [how to enable KEDA scaling](https://docs.microsoft.com/en-us/azure/container-apps/scale-app#keda-scalers-conversion).

<p align="center">
<img src="/images/devisland/article88/assets/azure-hints-03-containers-for-azure-devops-automation-11.png?raw=true" alt="Image not found"/>
</p>

Why we talk about KEDA and how it is realted to Azure DevOps Self-Hosted Agents? We talk about it because with KEDA we can scale Azure Container Apps based on agent pool queues for Azure Pipelines. I encourage you to read whole article about this topic [here](https://keda.sh/blog/2021-05-27-azure-pipelines-scaler/#placeholder-agent).

Below code presrnts Azure Container App definition using Azure Bicep with enabled KEDA auto-sclaing when there are more queued pipelines in Azure DevOps:

 ```bicep
resource containerApp 'Microsoft.App/containerApps@2022-03-01' = {
  name: contianerAppName
  tags: {
    'environment': environmentType
  }
  location: location
  properties: {
    managedEnvironmentId: containerAppEnvironmentId
    configuration: {
      activeRevisionsMode: 'single'
      secrets: [
        {
          name: 'acr-password'
          value: registryPassword
        }
        {
          name: 'az-devops-pat'
          value: azDevOpsPersonalAccessToken
        }
        {
          name: 'az-devops-org'
          value: azDevOpsOrganizationUrl
        }
      ]
      registries: [
        {
          server: registry
          username: registryUsername
          passwordSecretRef: 'acr-password'
        }
      ]
    }
    template: {
      revisionSuffix: revisionSuffix
      containers: [
        {
          resources: {
            cpu: json('1.75')
            memory: '3.5Gi'
          }
          image: containerImage
          name: contianerAppName
          env: envVars
        }
      ]
      scale: {
        minReplicas: 1
        maxReplicas: 10
        rules: [
          {
            name: contianerAppName
            custom: {
              type: 'azure-pipelines'
              metadata: {
                poolName: azDevOpsAgentPoolName
                targetPipelinesQueueLength: '1'
              }
              auth: [
                {
                  secretRef: 'az-devops-pat'
                  triggerParameter: 'personalAccessToken'
                }
                {
                  secretRef: 'az-devops-org'
                  triggerParameter: 'organizationURL'
                }
              ]
            }
          }
        ]
      }
    }
  }
}
```

Let's focus on the custom scaling rule with KEDA:


 ```bicep
resource containerApp 'Microsoft.App/containerApps@2022-03-01' = {
  name: contianerAppName
...
      scale: {
        minReplicas: 1
        maxReplicas: 10
        rules: [
          {
            name: contianerAppName
            custom: {
              type: 'azure-pipelines'
              metadata: {
                poolName: azDevOpsAgentPoolName
                targetPipelinesQueueLength: '1'
              }
              auth: [
                {
                  secretRef: 'az-devops-pat'
                  triggerParameter: 'personalAccessToken'
                }
                {
                  secretRef: 'az-devops-org'
                  triggerParameter: 'organizationURL'
                }
              ]
            }
          }
        ]
      }
    }
  }
}
```
As you can see, Azure Container Apps can be scaled basing on *targetPipelinesQueueLength* parameter value - the amount of pending jobs in the queue to scale on. It means that we can create more pods with Self-Hosted Agents when needed.

Where is the tricky part here?
You can run the agents with KEDA as a *Deployment* or a *Job* and scale them accordingly with a *ScaledObject* or a *ScaledJob*.

The case is that at the moment of writing this article, Azure Container Apps with KEDA supports only *ScaledObject*. The problem is when container apps scale down, they can stop any instance, including an long running agent pipeline. When running your agents as a deployment you have no control on which pod gets killed when scaling down. Using a *ScaledJob* is the preferred way to autoscale your Azure Pipelines agents if you have long running jobs. I recommend you to read more details in [this article](https://keda.sh/blog/2021-05-27-azure-pipelines-scaler/#placeholder-agent).

You can also see that at the moment of writing this article there is opened Issue by Jeff Hollan on GitHub to [Support the KEDA ScaledJobs / Jobs Pattern](https://github.com/microsoft/azure-container-apps/issues/24).

Anyway, we can still host multiple Azure DevOps Self-Hosted Agents with Azure Container Apps and deploy to resources in the Azure Virtual Network because of the fact that you can create Container Apps Environment in an existing [Azure Virtual Network](https://docs.microsoft.com/en-us/azure/container-apps/vnet-custom?tabs=bash&pivots=azure-portal).

**Cost**


## Azure Kubernetes Service

aaa


# Summary

In this article,...