---
title: "Docker and Azure Kubernetes Service for .NET Developers"
excerpt: "In this article I would like to describe some concepts around containerized ASP .NET Core applications, Docker, Azure Container Registry and Azure Kubernetes Service."
header:
  image: /images/devisland/article9/assets/aksintro01.png
---

<p align="center">
<img src="/images/devisland/article9/assets/aksintro01.png?raw=true" alt="Docker and Azure Kubernetes Service for .NET Developers"/>
</p>


<h3><strong>Short introduction</strong></h3>
I planned to write this article some time ago but this topic is so big that I did not how to start and how to collect all valuable details I learned. Finally - Docker and Azure Kubernetes Service for .NET Developers article is ready. Application containerization is not young concept but tools and new capabilities around this topic are still fresh and worth to track. In this article I would like to describe some concepts around containerized ASP .NET Core applications, Docker, Azure Container Registry and Azure Kubernetes Service. I hope that this content will help not only .NET Developers but also everyone who would like to understand the whole concept of containers.
<h3><strong>Prerequisites</strong></h3>
Before moving forward there are few prerequisites. First of all I would like to notice that I used Windows to prepare all the stuff presented below.

There are some tools you have to install before:

<strong>●</strong> Install Docker for Windows <a href="https://docs.docker.com/docker-for-windows/install/" target="_blank" rel="noopener">available here</a> - with Docker installed you can run containers locally on your machine

<strong>● </strong>Install Azure Command Line Interface <a href="https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest" target="_blank" rel="noopener">available here</a> - with Azure CLI you are able to connect to your Azure subscription and manage its resources through the command line

<strong>● </strong>Visual Studio 2017 <a href="https://visualstudio.microsoft.com/downloads" target="_blank" rel="noopener">available here</a> - you will use Visual Studio 2017 to create simple ASP .NET Core Web API application.
<h3><strong>ASP .NET Core</strong></h3>
<p style="text-align:center;"><img class="alignnone size-full wp-image-987" src="/images/devisland/article9/assets/aspimg.png" alt="" width="300" height="226" /></p>
Let me start from simple ASP .NET Core Web API application. I suppose that most of you had chance to create simple web application using this technology. In this article I will use basic template from Visual Studio.

<strong>IMPORTANT</strong>

Before this part make sure that Docker for Windows is running:

<img class="size-full wp-image-918 aligncenter" src="/images/devisland/article9/assets/aks6_6.png" alt="" width="119" height="162" />

Make sure that Docker is running Linux containers:

<img class="size-full wp-image-922 aligncenter" src="/images/devisland/article9/assets/aks6_7.png" alt="" width="300" height="215" />

1. Open Visual Studio 2017 and create new ASP .NET Core Web Application:

<img class=" wp-image-897 aligncenter" src="/images/devisland/article9/assets/aks1.png?w=300" alt="" width="656" height="455" />

2. When choosing default template select "API" and remember to select "Enable Docker Support". As Operating System (OS) select Linux - simple reason why is that containers on Linux are more mature:

<img class="wp-image-900 aligncenter" src="/images/devisland/article9/assets/aks2.png?w=300" alt="" width="669" height="471" />

Once solution is created you should see additional parts:

<strong>● </strong>docker-compose

<strong>● </strong>Dockerfile

<img class=" wp-image-904 aligncenter" src="/images/devisland/article9/assets/aks3_1.png?w=271" alt="" width="472" height="523" />

I will describe them in details in the next section so for now I will only mention that they are required to run our application in Docker container.

Plese note that at the bottom bar you have option to run your application with Docker. Click it and after few seconds browser should be displayed with two default values returned from Values controller in our Web API:

<img class=" wp-image-907 aligncenter" src="/images/devisland/article9/assets/aks4.png?w=300" alt="" width="726" height="162" />

<strong>Congratulations!</strong> You just deployed and started application in Docker container!

<img class=" wp-image-911 aligncenter" src="/images/devisland/article9/assets/aks5.png?w=300" alt="" width="633" height="194" />

Once you started application, Visual Studio created Docker image for you using Docker tools. You can verify this. Open Azure CLI or PowerShell and type: <em>docker images</em> :

<img class=" wp-image-914 aligncenter" src="/images/devisland/article9/assets/aks6.png?w=300" alt="" width="641" height="335" />

This first time when you managed to start Asp .NET Core web app with Docker... Great but is it all about containers? Am I ready to start creating highly scalable applications using containers? Not really... Not yet.
<h3><strong>Docker</strong></h3>
<img class="alignnone wp-image-982 aligncenter" src="/images/devisland/article9/assets/dockerimg.png?w=300" alt="" width="365" height="365" />

In above sample we used Docker - but what Docker exactly is and what are main concepts connected with it?

<strong>Docker</strong>

An open, containerization platform for developers and sysadmins to build, ship, and run distributed applications. It enables to package and deploy an application or service as an isolated unit containing all of its dependencies  whether on laptops, data center VMs, or the cloud.

<img class=" wp-image-937 aligncenter" src="/images/devisland/article9/assets/aks6_81.png?w=300" alt="" width="446" height="442" />

&nbsp;

<strong>Container image</strong>

Container image an be explained as a box with all the dependencies and information required to create a container instance. Image is defined by the Docker file (described below) and it becomes immutable once built. Images can inherit configuration from other images - so the same as in .NET class can extend another class. It is much easier to create new image without starting from scratch. For instance for the application we created on the beginning of this article there was ASP .NET Core base image used, create by Microsoft. Open Dockerfile from the project and you should notice below line:

<em>FROM microsoft/aspnetcore:2.0 AS base</em>

<strong>Container</strong>

Docker container is an instance of a Docker image and represents the execution of a single application, process, or service. You can create multiple container instances from the same image. In our case we runned one container with ASP .NET Core Web API application.

<strong>Container image tag</strong>

Docker images can be tagged. You can apply labels to images to differentiate them. For our ASP .NET Core Web API application image Visual Studio applied "dev" tag (you can see screenshot from the console above).

<strong>Container host</strong>

Docker host is the underlying Operating System (OS) on which you will run Docker containers. It can be Physical or Virtual computer system. Docker will utilize shared OS kernel resources to run containers. Currently there are two types of Docker hosts: Linux and Windows.

<strong>Dockerfile</strong>

A text file that contains instructions for how to build a Docker image. Below there is Dockerfile from our sample ASP .NET Core Web API project:

```
FROM microsoft/aspnetcore:2.0 AS base
WORKDIR /app
EXPOSE 80

FROM microsoft/aspnetcore-build:2.0 AS build
WORKDIR /src
COPY Sample.WebAPI/Sample.WebAPI.csproj Sample.WebAPI/
RUN dotnet restore Sample.WebAPI/Sample.WebAPI.csproj
COPY . .
WORKDIR /src/Sample.WebAPI
RUN dotnet build Sample.WebAPI.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish Sample.WebAPI.csproj -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "Sample.WebAPI.dll"]
}
```

As I mentioned before  Docker image can base on the other Docker images so you can choose a base image that contains the elements you need, and then copy in your own application on top. In our case ASP .NET Core base images provided by Microsoft was used.

<strong>Docker Compose</strong>

Docker compose YAML file specifies one or more containers that make up a single application or system. Within this file you have to specify the images that need to be started inside Docker containers, what are their dependencies or under which ports they should be available. Sample Docker Compose file content looks like below (taken from our simple Web API project). Docker Compose is also a command-line tool so using a single command (docker-compose up) you can deploy the whole multi-container application.

```
version: '3.4'

services:
  sample.webapi:
    image: ${DOCKER_REGISTRY}samplewebapi
    build:
      context: .
      dockerfile: Sample.WebAPI/Dockerfile
```

<strong>Images repository</strong>

Images repository contains Docker images collection labeled with a tag that indicates the image version.

<strong>Registry</strong>

Registry is a service that provides access to images repositories. The most popular Docker registry is <a href="https://hub.docker.com" target="_blank" rel="noopener">Docker Hub</a> which provides access to many different image repositories. Another example can be Azure Container Registry described below in the article.

<strong>Docker glossary</strong>

In this section I would like to present and describe some helpful Docker commands. Use them either in Azure CLI or in PowerShell.

<em><span class="TextRun SCXO264619777" lang="EN-US" xml:lang="EN-US"><span class="NormalTextRun SCXO264619777"><strong>●</strong> docker images</span></span></em><span class="EOP SCXO264619777"> - Display all Docker images available on host</span>

<span class="EOP SCXO264619777"><em><strong>●</strong> docker build PATH_TO_DOCKERFILE</em> - Build Docker image from the Dockerfile</span>

<em><strong>●</strong> docker rmi IMAGE_NAME</em> - Remove Docker image with specific name

<em><strong>●</strong> docker rmi [IMAGE_NAME]:[TAG_NAME]</em> - Remove Docker image with specific name and tag

<em><strong>●</strong> docker tag [IMAGE_NAME]:[TAG]</em> - Tag Docker image

<em><strong>●</strong></em> <em>docker-compose build</em> - Build the images but do not start Docker container instances

<em><strong>●</strong></em> <em>docker-compose up</em> - Build the images if they do not exist and start containers instances

<em><strong>●</strong></em> <em>docker-compose up --build</em> -  Build images even they already exist and then start container instances

<em><strong>●</strong></em>  <span class="TextRun SCXO3463294" lang="EN-US" xml:lang="EN-US"><span class="NormalTextRun SCXO3463294"><em>docker push</em> - Push an image or a repository to a registry</span></span>
<h3><strong>Azure Container Registry (ACR)</strong></h3>
<img class=" wp-image-980 aligncenter" src="/images/devisland/article9/assets/aks7_1.png" alt="" width="293" height="243" />

Azure Container Registry is cloud container registry available on Microsoft Azure cloud platform for storing and managing Docker container images but not only - ACR allows you to store images for all types of container deployments including DC/OS, Docker Swarm, Kubernetes, and Azure services such as App Service, Batch, Service Fabric.
<h3><strong>Azure Kubernetes Service (AKS)</strong></h3>
<p style="text-align:center;"><img class="alignnone wp-image-983" src="/images/devisland/article9/assets/kubernetesimg.png?w=300" alt="" width="328" height="169" /></p>
Before talking about AKS is worth to describe what Kubernetes exactly is.

<strong>Kubernetes</strong>

Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications. It is also called "containers orchestrator" because it enables managing groups of running containers and their lifecycle.

It gives the freedom to take advantage of on-premises, hybrid, or public cloud infrastructure - so in this case Microsoft Azure cloud. I encourage you to read more about Kubernetes on the official <a href="https://kubernetes.io/" target="_blank" rel="noopener">website</a>.

Below I described some Kubernetes concepts:

<strong>Namespace</strong>

Namespace can be compared to namespace from .NET world. Each namespace is isolated. Each application (or micro-service) can be deployed to specific namespace. Kubernetes cluster has at least two built-in namespaces. These are default and kube-system.

<strong>Node</strong>

Node is a host withing a Kubernetes cluster - either physical or virtual machine. Nodes are managed by master - a collection of components, such
as API server, scheduler, and controller manager. You can increase or decrease number of nodes withing Kubernetes cluster.

<strong>Cluster</strong>

Kubernetes cluster consists of nodes ( physical or virtual machines that run applications and cloud workflows).

<strong>Pod</strong>

Kubernetes Pod hosts application instance. A Pod is a Kubernetes abstraction that represents a group of one or more application containers (in our case Docker) and some shared resources for those containers like: shared storage or networking. A Pod models an application-specific "logical host".

<strong>Service</strong>

Kubernetes Service is an abstraction which defines a logical set of Pods including policy by which to access them. To clarify and not make topic so complicated imagine that service is just an abstraction which enables decoupling between pods and external world and access to applications within containers in the pod. For instance you can define port number or transport protocol.

<strong>ReplicaSet</strong>

ReplicaSet manages a group of pods of the same type and ensures that a specific number of pods are always running. If a pod crashes, a replica set restores it.

<strong>Deployment</strong>

Deployment is a resource that ensures the reliable roll out of an application. It creates a replica set that is used to manage a group of pods of the same type. If a pod crashes then another one is automatically created through
the replica set.

<strong>Ingress</strong>

Ingress is load balancer which enables exposition one or more services to the outside world providing externally visible URLs to services and
load-balance traffic with SSL termination. Ingresses can also be used to define network routes between namespaces and pods in conjunction with network policies. Ingress Controller managing them with actions such as
requests limitation or  URLs redirection.

<strong>ConfigMap</strong>

ConfigMap enables keeping image configuration options separate from containers and pods. Configuration options are stored as a key-value pairs and exposed as an environment variables.

<strong>Secret</strong>

Secrets are very similar to ConfigMaps but they are responsible for storing sensitive information like passwords.
<h4><strong>Azure Kubernetes Service (AKS)</strong></h4>
Azure Kubernetes Service enables reduction of the complexity and operational overhead of managing Kubernetes. With AKS you do not have to create and configure each part of Kubernetes manually - Azure handles critical tasks like health monitoring  or maintenance. It is worth to mention that AKS service is free, you only pay for the agent nodes within the clusters.

There are few additional benefits from using Azure Kubernetes Service:

<em><strong>● </strong></em>Kubernetes master and all nodes are deployed and configured

<em><strong>● </strong></em>Azure Kubernetes Service supports the Docker image format

<em><strong>● </strong></em>AKS clusters are created with support for Azure Files and Azure Disks

<em><strong>● </strong></em>Integration with with Azure Container Registry (ACR)

<em><strong>● </strong></em>HTTP Application Routing solution makes it easy to access applications deployed to AKS cluster

To sum up it is much easier to configure and use Kubernetes with Azure Kubernetes Service.

<strong>Kubernetes glossary</strong>

In this section I would like to describe some concepts of Kubernetes and present some helpful commands. Use them either in Azure CLI or in PowerShell.

<em><span class="TextRun SCXO264619777" lang="EN-US" xml:lang="EN-US"><span class="NormalTextRun SCXO264619777"><strong>●</strong> kubectl</span></span></em><span class="EOP SCXO264619777">- command line interface for running commands against Kubernetes clusters</span>

<em><span class="TextRun SCXO264619777" lang="EN-US" xml:lang="EN-US"><span class="NormalTextRun SCXO264619777"><strong>●</strong> kubectl get pods </span></span></em><span class="EOP SCXO264619777">- command to display all pods in the default namespace</span>

<em><span class="TextRun SCXO264619777" lang="EN-US" xml:lang="EN-US"><span class="NormalTextRun SCXO264619777"><strong>●</strong> kubectl apply -f sample-app-microservice.yaml </span></span></em><span class="EOP SCXO264619777">- command to deploy new service</span>

<em><span class="TextRun SCXO264619777" lang="EN-US" xml:lang="EN-US"><span class="NormalTextRun SCXO264619777"><strong>●</strong> kubectl get deployment </span></span></em><span class="EOP SCXO264619777">- command to display deployments in the default namespace</span>

<em><span class="TextRun SCXO264619777" lang="EN-US" xml:lang="EN-US"><span class="NormalTextRun SCXO264619777"><strong>●</strong> kubectl apply -f sample-configmap.yaml </span></span></em><span class="EOP SCXO264619777">- command to create new ConfigMap from YAML file</span>
<h3><strong>All together</strong></h3>
Now when you have some bigger knowledge it is time to connect the dots. For the purpose of this article I prepared configuration instructions so you can start using Azure Kubernetes Service together with Docker containers and Azure Container Registry. We will use simple ASP .NET Core Web API application we created on the beginning of this article. It will be running inside Docker container orchestrated by Kubernetes.

<strong>Create Azure Container Registry</strong>

1. Open Azure portal and type "ACR" in the "New Resource" window:

<img class=" wp-image-1035 aligncenter" src="/images/devisland/article9/assets/aks7.png?w=300" alt="" width="572" height="330" />

2. Select "Container Registry":

<img class=" wp-image-1037 aligncenter" src="/images/devisland/article9/assets/aks8.png?w=300" alt="" width="563" height="334" />

3. Type the name of your registry and resource group. SKU can be set to "Basic". Then click "Create":

<img class=" wp-image-1039 aligncenter" src="/images/devisland/article9/assets/aks9.png?w=150" alt="" width="297" height="596" />

4. Once ACR is created you can see it in the dashboard:

<img class=" wp-image-1042 aligncenter" src="/images/devisland/article9/assets/aks10.png?w=300" alt="" width="622" height="230" />

5. Click it and go to "Repositories" tab. You should see empty list:

<img class=" wp-image-1045 aligncenter" src="/images/devisland/article9/assets/aks11.png?w=300" alt="" width="624" height="366" />

6. It is time to add application Docker image to ACR. To achieve it you have to use Azure CLI and sign in to your Azure subscription:

<img class=" wp-image-1049 aligncenter" src="/images/devisland/article9/assets/aks12.png?w=300" alt="" width="617" height="273" />

You can sign in with this command:

<em>az login --tenant &lt;&lt;YOUR_TENANT_HERE&gt;&gt;</em>

Once you authenticate you have access to your Azure resources.

7. Before we push the image to Azure Container Registry we have to tag it. We will use "dev" tag. You can tag your image with this command:

<em>docker tag [Image ID] [ACR_NAME].azureacr.io/microservices/samplewebapi:dev</em>

<img class="alignnone wp-image-1048 aligncenter" src="/images/devisland/article9/assets/aks13.png?w=300" alt="" width="591" height="284" />

8. Login to ACR using below command:

<em>az acr login --name [ACR_NAME]</em>

<img class=" wp-image-1056 aligncenter" src="/images/devisland/article9/assets/aks14_1.png?w=300" alt="" width="579" height="203" />

9. Push image to ACR with below command:

<em>docker push [ACR_NAME].azure.io/microservices/samplewebapi:dev</em>

<img class=" wp-image-1053 aligncenter" src="/images/devisland/article9/assets/aks14.png?w=300" alt="" width="597" height="312" />

Now go to Azure portal and verify if there is new image with "dev" tag for your application:

<img class=" wp-image-1058 aligncenter" src="/images/devisland/article9/assets/aks15.png?w=300" alt="" width="583" height="224" />
<p style="text-align:center;"><img class="alignnone wp-image-1059" src="/images/devisland/article9/assets/aks16.png?w=300" alt="" width="582" height="147" /></p>
Image is ready - it can be pulled from Kubernetes so lets see how to configure AKS below.

<strong>Create Azure Kubernetes Service</strong>

1. Open Azure portal and type "AKS" in the "New Resource" window:

<img class=" wp-image-1061 aligncenter" src="/images/devisland/article9/assets/aks19.png?w=300" alt="" width="569" height="306" />

2. Select "Kubernetes Service":
<p style="text-align:center;"><img class="alignnone wp-image-1063" src="/images/devisland/article9/assets/aks20.png?w=300" alt="" width="570" height="188" /></p>
3. Now setup AKS accordingly to below screens:

In the first step you have to provide some important details like cluster name, resource group where it will be created, region, Kubernetes version, node size and node count:
<p style="text-align:center;"><img class="alignnone wp-image-1065" src="/images/devisland/article9/assets/aks21.png?w=300" alt="" width="649" height="640" /></p>
Next section should look like below. We want basic network configuration and no HTTP application routing:
<p style="text-align:center;"><img class="alignnone wp-image-1066" src="/images/devisland/article9/assets/aks22.png?w=280" alt="" width="650" height="696" /></p>
It is good to enable Azure Monitor to measure Kubernetes cluster performance:
<p style="text-align:center;"><img class="alignnone wp-image-1067" src="/images/devisland/article9/assets/aks23.png?w=273" alt="" width="649" height="713" /></p>
There will be no tags:
<p style="text-align:center;"><img class="alignnone wp-image-1068" src="/images/devisland/article9/assets/aks24.png?w=293" alt="" width="652" height="667" /></p>
Verification and summary:
<p style="text-align:center;"><img class="alignnone wp-image-1069" src="/images/devisland/article9/assets/aks25.png?w=300" alt="" width="649" height="630" /></p>
Click "Create" button to create AKS cluster. After few minutes you should see the result. Please open resource groups blade - note that there are two additional resource groups created. Select name of the resource group you set during configuration:

<img class=" wp-image-1071 aligncenter" src="/images/devisland/article9/assets/aks26.png?w=300" alt="" width="577" height="263" />
<p style="text-align:center;"><img class="alignnone wp-image-1072" src="/images/devisland/article9/assets/aks27.png?w=300" alt="" width="582" height="355" /></p>
Select cluster - you should see main tab with "View Kubernetes dashboard" section:

<img class=" wp-image-1084 aligncenter" src="/images/devisland/article9/assets/aks28.png?w=300" alt="" width="583" height="167" />

Once you select it three commands will be presented:

<em><strong>● </strong>az aks install-cli </em>- enables you to install Kubernetes tools so you can access Kubernetes cluster from Azure CLI

<em>● az aks get-credentials --resource-group devisland-aks-rg --name devisland-aks </em>- Get credentials so you can access Kubernetes cluster

<em>● az aks browse --resource-group devisland-aks-rg --name devisland-aks </em>- open dashboard in browser

<img class=" wp-image-1089 aligncenter" src="/images/devisland/article9/assets/aks29.png?w=300" alt="" width="574" height="501" />

Once you apply last command Kubernetes dashboard will appear:

<img class=" wp-image-1092 aligncenter" src="/images/devisland/article9/assets/aks30.png?w=300" alt="" width="569" height="360" />

<strong>Setup connection between Azure Kubernetes Service and Azure Container Registry</strong>

We have to configure access to ACR from Kubernetes cluster. To do it we can setup secret. Remember that you have to be logged in to your Azure subscription through Azure CLI first.

1. Open ACR blade in Azure portal and go to "Keys" tab. We will use this information in the next step (copy registry name, username and password):

<img class=" wp-image-1102 aligncenter" src="/images/devisland/article9/assets/aks16_1.png?w=300" alt="" width="563" height="469" />

2. In Azure CLI type below command:

<em>kubectl create secret docker-registry [ACR_NAME]-connection --docker-server=[ACR_NAME] --docker-username=[ACR_NAME] --docker-password=[PASSWORD_FROM_THE_PORTAL] --docker-email=[YOUR_EMAIL]</em>

This command creates ACR secret in Kubernetes so it can access ACR repository with Docker images.

3. Get ServiceAccount.yml file from Kubernetes with below command:

kubectl get serviceaccounts default -o yaml &gt; ./serviceaccount.yml

ServiceAccount YAML file contains configuration for pods used in Kubernetes. They are tied to a set of credentials stored as Secrets.

At the end of the file add imagePullSecrets section to provide access to ACR secret:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2018-06-30T20:17:46Z
  name: default
  namespace: default
  resourceVersion: "141"
  selfLink: /api/v1/namespaces/default/serviceaccounts/default
  uid: a7385849-7ca2-11e8-85bd-0a58ac1f0f54
secrets:
- name: default-token-txw4m
imagePullSecrets:
- name: devislandacr-connection
```

Update file on Kubernetes using below command:

<em>kubectl replace serviceaccount default -f ./serviceaccount.yml</em>

<img class=" wp-image-1136 aligncenter" src="/images/devisland/article9/assets/aks37_2.png?w=300" alt="" width="695" height="234" />

Once you created static IP save it somewhere - we will use it soon.

<strong>Configure NGINX ingress loadbalancer</strong>

Ingress controller is using "ingress resource" with all micro services (APIs) mapping so we can have many pods and ingress will handle connection redirection to them. We are using <a href="https://github.com/kubernetes/ingress-nginx" target="_blank" rel="noopener">NGINX.</a>

With ingress we do not have to create separate IP for each service in Kubernetes.

There are few ways to install ingress - in this case I am using Helm.

<strong>Helm</strong>

<img class="size-medium wp-image-1115 aligncenter" src="/images/devisland/article9/assets/helm.png?w=207" alt="" width="207" height="300" />

Helm is package manager for Kubernetes like NuGet in Visual Studio. We are using Helm to install nginx ingress controller in Kubernetes cluster.

<a href="https://docs.helm.sh/using_helm/#install" target="_blank" rel="noopener">Download</a> Helm. I used Helm application for Windows. To use helm you have to switch directory in Azure CLI to directory where Helm application file is located.

Sign in to Kubernetes cluster in Azure CLI first (using command from the Azure portal I presented above) and then apply below command:

<em>helm init</em>

Helm has two major components:

<em><strong>● </strong></em>Helm Client is a command-line client for end users

● Tiller Server is an in-cluster server that interacts with the Helm client, and interfaces with the Kubernetes API server

Once Helm is ready we can install NGINX with below command:

<em>helm install stable/nginx-ingress --name [NAME_UP_TO_YOU]-nginx --set rbac.create=false</em>

You should see that ingress was installed with success:

<img class=" wp-image-1130 aligncenter" src="/images/devisland/article9/assets/aks36.png?w=300" alt="" width="725" height="314" /><img class=" wp-image-1119 aligncenter" src="/images/devisland/article9/assets/aks37.png?w=300" alt="" width="708" height="408" />

&nbsp;

<strong>Verify public IP to access Kubernetes microservices from the Internet</strong>

If we want to access our Web API hosted as microservice in Kubernetes we have to check public static IP. You can find it in the resource group which starts with "MC...":

<img class=" wp-image-1141 aligncenter" src="/images/devisland/article9/assets/aks43.png?w=300" alt="" width="691" height="30" />

<img class=" wp-image-1143 aligncenter" src="/images/devisland/article9/assets/aks42.png?w=300" alt="" width="690" height="494" />

&nbsp;

<strong>Update YAML files</strong>

Now we have to update ingress resource (ingress.yaml file) so ingress controller knows the mapping of different micro-services (in our case one with sample Web API application):

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: devisland-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        backend:
          serviceName: devisland-sample-webapi-service
          servicePort: 8080
```

Use below command to apply changes. Remember to change "serviceName" if you chose different:

kubectl apply -f ingress.yaml

<img class=" wp-image-1128 aligncenter" src="/images/devisland/article9/assets/aks33.png?w=300" alt="" width="767" height="332" />

Then we have to update ingress service file with our public static IP address obtained earlier:

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx-ingress
    chart: nginx-ingress-0.22.0
    component: controller
    heritage: Tiller
    release: devisland-nginx
  name: devisland-nginx-nginx-ingress-controller
  selfLink: /api/v1/namespaces/default/services/devisland-nginx-nginx-ingress-controller
spec:
  externalTrafficPolicy: Local
  loadBalancerIP: 104.40.191.248
  ports:
  - name: http
    nodePort: 30781
    port: 80
    protocol: TCP
    targetPort: http
  - name: https
    nodePort: 32026
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app: nginx-ingress
    component: controller
    release: devisland-nginx
  sessionAffinity: None
  type: LoadBalancer
```

kubectl apply -f ingress-service.yaml

Then we have to create Service for our Web API application with sample-web-service-api.yaml file:

```csharp
apiVersion: v1 
kind: Service 
metadata: 
  name:  devisland-sample-webapi-service
  namespace: default
spec: 
  type: NodePort 
  selector: 
    app:  devisland-sample-webapi-app
  ports: 
  - name: http 
    port: 8080 
    targetPort: 80
```

kubectl apply -f sample-web-service-api.yaml

Once we defined Service for our Web API app we have to create Deployment with sample-webapi-deployment.yaml file. Note that in this file we are defining number of pods and from which container registry Docker image should be pulled:

```csharp
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: sample-webapi-deployment
  namespace: default
spec:
  selector:
    matchLabels:
      app: devisland-sample-webapi-app
  replicas: 2 # tells deployment to run 2 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: devisland-sample-webapi-app
    spec:
      containers: 
      - name: devisland-sample-webapi-container 
        image: devislandacr.azurecr.io/microservices/samplewebapi:dev 
        imagePullPolicy: Always
        ports:
        - containerPort: 80
```

<img class=" wp-image-1133 aligncenter" src="/images/devisland/article9/assets/aks39.png?w=300" alt="" width="685" height="205" />

Open Kubernetes dashboard and verify result:

<img class=" wp-image-1126 aligncenter" src="/images/devisland/article9/assets/aks40.png?w=300" alt="" width="703" height="344" />
<h3><strong>Test with Postman</strong></h3>
Open Postman and type IP addess and path to your sample Web API:

<img class=" wp-image-1139 aligncenter" src="/images/devisland/article9/assets/aks41.png?w=300" alt="" width="701" height="154" />
<h3><strong>Wrapping up </strong></h3>
The end.

I hope that this article helped you understand some concepts related with Docker and Kubernetes. We discussed how to create Docker image, how to push it to the Azure Container Registry and how to run ASP .NET Core Web API app inside Kubernetes pod and Docker container. I encourage you to do all the steps by yourself so you can understand clearly what is happening in each of them. Good luck!