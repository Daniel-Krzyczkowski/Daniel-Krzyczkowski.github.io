---
title: "DevSecOps on Azure - part6: Deploy securely to Azure resources in the Virtual Network"
excerpt: "This article presents the approaches to securely deploy to Azure resources in the Virtual Network"
header:
  image: /images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-01.png
---

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-01.png?raw=true" alt="DevSecOps on Azure - part6"/>
</p>

# Introduction

This is the next article from the series called *DevSecOps practices for Azure cloud workloads*. In this article, I would like to talk about an important topic that is not always obvious, especially for the beginning Azure Cloud Engineers. How to deploy code to Azure resources (like Azure Function Apps or Azure Container Apps) integrated and isolated with Azure Virtual Network.

**I also strongly recommend reading my other article which is strongly related to this topic:** [*Azure Hints Series - Containers for Azure DevOps Automation*](https://techmindfactory.com/Containers-For-Azure-DevOps-Automation/)

# Azure cloud resources with, and without Virtual Network integration

Before we continue talking about secure deployments, let's stop for a minute and understand the important fact about Azure cloud resources. Resources like Azure Functions, Azure Web Apps, or Azure Container Apps can be created without Azure Virtual Network integration. It does not mean that there is no network infrastructure underneath because of course, there is. The thing is that we can create all these resources without additional network-level isolation, integration, and security.

Let me first put an example. Below we can see two solutions. The first one is without Azure Virtual Network integration, the second utilizes Azure Virtual Network to isolate public access to Azure resources:


<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-02.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-03.png?raw=true" alt="Image not found"/>
</p>

The most important difference is that in the first solution we do not have any additional layer of security around network access to Azure resources. Of course, Azure has mechanisms to detect potential attacks like Basic DDoS Protection protection at no additional charge but this is not the only option that makes our solution fully secure.

When we look into Azure Security Benchmark - [*Security Control v3: Network security*](https://learn.microsoft.com/en-us/security/benchmark/azure/security-controls-v3-network-security#ns-2-secure-cloud-services-with-network-controls) we will find out below recommendations:

#### [NS-1: Establish network segmentation boundaries](https://learn.microsoft.com/en-us/security/benchmark/azure/security-controls-v3-network-security#ns-1-establish-network-segmentation-boundaries)

*Create a virtual network (VNet) as a fundamental segmentation approach in your Azure network, so resources such as VMs can be deployed into the VNet within a network boundary. To further segment the network, you can create subnets inside VNet for smaller sub-networks.*
*Use network security groups (NSG) as a network layer control to restrict or monitor traffic by port, protocol, source IP address, or destination IP address.*

#### [NS-2: Secure cloud services with network controls](https://learn.microsoft.com/en-us/security/benchmark/azure/security-controls-v3-network-security#ns-2-secure-cloud-services-with-network-controls)

*Deploy private endpoints for all Azure resources that support the Private Link feature, to establish a private access point for the resources. You should also disable or restrict public network access to services where feasible.*
*For certain services, you also have the option to deploy VNet integration for the service where you can restrict the VNET to establish a private access point for the service.*


#### [NS-6: Deploy web application firewall](https://learn.microsoft.com/en-us/security/benchmark/azure/security-controls-v3-network-security#ns-6-deploy-web-application-firewall)

*Use web application firewall (WAF) capabilities in Azure Application Gateway, Azure Front Door, and Azure Content Delivery Network (CDN) to protect your applications, services and APIs against application layer attacks at the edge of your network. Set your WAF in "detection" or "prevention mode", depending on your needs and threat landscape.*


As we can read above, it is highly recommended to utilize Azure Virtual Network to enhance the security of solutions built on the Azure cloud. This is why in the second architecture diagram I included components like:
1. Azure Virtual Network
2. Azure Private Links
3. Azure Private DNS
4. Azure Front Door with WAF


# Azure Virtual Network and DevOps automation

Once we isolate all resources and we start utilizing integration with Azure Virtual Network, we can be surprised that deployments from Azure DevOps, GitHub, or any other automation tool will stop working. Here is a nice example of what will happen when we isolate Azure Container Registry using Private Link and try to push Docker images from Azure DevOps:

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-04.png?raw=true" alt="Image not found"/>
</p>

This is because we isolated our Azure resources. This will happen to every deployment to Azure resource isolated with Azure Virtual Network. This is because Azure DevOps agents or GitHub runners are not able to connect to these resources. In this case, we have to verify if we can update firewall rules for specific services or utilize a self-hosted CI/CD agent. Let me put a specific example here.

#### Azure Container Registry

Azure Container Registry (ACR) supports Private Links so we can disable public access to it:

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-05.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-06.PNG?raw=true" alt="Image not found"/>
</p>

It means that we can access ACR only from inside the Azure Virtual Network. What about the situation when we want to build and push Docker images to ACR using GitHub-hosted runners or Azure DevOps agents? In such a scenario we can update the firewall of ACR dynamically to enable temporary access only from the specific IP address. In this case, this will be the CI/CD agent IP address. To make it more clear, here is the template I created for the Azure DevOps pipeline to dynamically get IP address of the agent, update ACR firewall to allow access from this IP address, and once Docker images are successfully pushed, IP address is removed, and public access is disabled:

```yaml
parameters:
- name: azureSubscriptionConnectionName
  type: string
- name: containerRegistryName
  type: string

jobs:
- job: Build
  displayName: 'Build Project'
  pool:
    vmImage: 'ubuntu-latest'
  steps: 
    - task: AzureCLI@2
      displayName: 'Add network rule to ACR'
      inputs:
          azureSubscription: ${{parameters.azureSubscriptionConnectionName}}
          scriptType: 'bash'
          scriptLocation: 'inlineScript'
          inlineScript: |
                IP=$(curl curl https://ifconfig.me/ip)
                az acr update --name acrtmfdevsecopsdev --public-network-enabled true
                az acr network-rule add \
                    --name acrtmfdevsecopsdev \
                    --ip-address $IP

    - template: ../tasks/build.docker.images.task.yml 
      parameters:
          azureSubscriptionConnectionName: ${{parameters.azureSubscriptionConnectionName}}
          containerRegistryName: ${{parameters.containerRegistryName}}

    - template: ../tasks/push.docker.images.task.yml 
      parameters:
          azureSubscriptionConnectionName: ${{parameters.azureSubscriptionConnectionName}}
          containerRegistryName: ${{parameters.containerRegistryName}}
    
    - task: AzureCLI@2
      displayName: 'Remove network rule from ACR'
      inputs:
          azureSubscription: ${{parameters.azureSubscriptionConnectionName}}
          scriptType: 'bash'
          scriptLocation: 'inlineScript'
          inlineScript: |
                IP=$(curl curl https://ifconfig.me/ip)
                az acr network-rule remove \
                    --name acrtmfdevsecopsdev \
                    --ip-address $IP
                az acr update --name acrtmfdevsecopsdev --public-network-enabled false
```

With the such solution, we can still utilize agents provided by Azure DevOps and GitHub. However, there can be more situations when we do not want to allow access to Azure resources from any IP addresses outside of our Azure Virtual Network or like for Azure Web Apps, when we enable Private Endpoints (Private Link) to Web App, all public access is disabled. In this case, the best solution is to utilize Azure DevOps self-hosted agents, or GitHub self-hosted runners.

## Options to host runners in the Azure cloud

In the scenario, when we have our Azure resources isolated in Azure Virtual Network, we can create self-hosted agents and runners utilizing one of the Azure services connected to our Azure Virtual Network like:

1. Azure Virtual Machines (which is costly)
2. Azure Container Instances (self-hosted runner/agent is hosted in Docker container)
3. Azure Container Apps (self-hosted runner/agent is hosted in Docker container)

It is always good to look at the cost. It is cheaper to utilize Docker to host self-hosted runners instead of using Virtual Machines but of course, it is important to assess each situation/environment individually because Virtual Machines can be helpful in some scenarios. I encourage you to read my article called [Azure Hints Series - Containers for Azure DevOps Automation](https://techmindfactory.com/Containers-For-Azure-DevOps-Automation/) where I explained different options (including the cost aspect) when it comes to hosting options. In this article, we will see how to utilize Docker for Azure DevOps self-hosted agents and GitHub self-hosted runners.


## GitHub Self-Hosted Runners

With GitHub self-hosted runners we can host our runners and customize the environment used to run jobs in GitHub Actions workflows. Self-hosted runners can be physical, virtual, in a container, on-premises, or in the cloud.

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-07.PNG?raw=true" alt="Image not found"/>
</p>

I encourage you to read more about GitHub self-hosted runners [in the official documentation](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners).

### Run GitHub self-hosted runner in Docker container

To run GitHub self-hosted runner we need two files:

1. Dockerfile with the definition of all tools we need in the container (like Azure CLI or PowerShell) and reference to *start.sh* script to start the runner.
2. start.sh - script to run self-hosted runner which will connect to our GitHub account/organization.

Here is the Dockerfile content, this will install Azure CLI together with PowerShell:

```Dockerfile
FROM ubuntu:20.04

#input GitHub runner version argument
ARG RUNNER_VERSION
ENV DEBIAN_FRONTEND=noninteractive

LABEL Author="Daniel Krzyczkowski"
LABEL GitHub="https://github.com/Daniel-Krzyczkowski"
LABEL BaseImage="ubuntu:20.04"
LABEL RunnerVersion=${RUNNER_VERSION}

# update the base packages + add a non-sudo user
RUN apt-get update -y && apt-get upgrade -y && useradd -m docker

# install Azure CLI and other required packages
RUN apt-get install -y --no-install-recommends \
    curl nodejs wget unzip vim git azure-cli jq build-essential libssl-dev libffi-dev python3 python3-venv python3-dev python3-pip

ARG PS_VERSION=7.1.4
ARG PS_PACKAGE=powershell_${PS_VERSION}-1.ubuntu.20.04_amd64.deb
ARG PS_PACKAGE_URL=https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/${PS_PACKAGE}
# Define ENVs for Localization/Globalization
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    # set a fixed location for the Module analysis cache
    PSModuleAnalysisCachePath=/var/cache/microsoft/powershell/PSModuleAnalysisCache/ModuleAnalysisCache \
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-Ubuntu-20.04


# Install dependencies and clean up
RUN apt-get clean

RUN apt-get update \
    && apt-get install --no-install-recommends -y \
    # curl is required to grab the Linux package
        curl \
    # less is required for help in powershell
        less \
    # requied to setup the locale
        locales \
    # required for SSL
        ca-certificates \
        gss-ntlmssp \
    # PowerShell remoting over SSH dependencies
        openssh-client \
    # Download the Linux package and save it
    && echo ${PS_PACKAGE_URL} \
    && curl -sSL ${PS_PACKAGE_URL} -o /tmp/powershell.deb \
    && apt-get install --no-install-recommends -y /tmp/powershell.deb \
    && apt-get dist-upgrade -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && locale-gen $LANG && update-locale \
    # remove powershell package
    && rm /tmp/powershell.deb \
    # intialize powershell module cache
    # and disable telemetry
    && export POWERSHELL_TELEMETRY_OPTOUT=1 \
    && pwsh \
        -NoLogo \
        -NoProfile \
        -Command " \
          \$ErrorActionPreference = 'Stop' ; \
          \$ProgressPreference = 'SilentlyContinue' ; \
          while(!(Test-Path -Path \$env:PSModuleAnalysisCachePath)) {  \
            Write-Host "'Waiting for $env:PSModuleAnalysisCachePath'" ; \
            Start-Sleep -Seconds 6 ; \
          }"


RUN pwsh -Command Install-Module AZ -Force


# cd into the user directory, download and unzip the github actions runner
RUN cd /home/docker && mkdir actions-runner && cd actions-runner \
    && curl -O -L https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz \
    && tar xzf ./actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz

# setup permissions for docker user
RUN chown -R docker ~docker && /home/docker/actions-runner/bin/installdependencies.sh

# add over the start.sh script
ADD script/start.sh start.sh

# make the script executable
RUN chmod +x start.sh

# set the user to "docker" so all subsequent commands are run as the docker user
USER docker

# set the entrypoint to the start.sh script
ENTRYPOINT ["./start.sh"]
```

Here is the start.sh script file:

```sh
#!/bin/bash

GH_OWNER=$GH_OWNER
GH_REPOSITORY=$GH_REPOSITORY
GH_TOKEN=$GH_TOKEN

RUNNER_SUFFIX=$(cat /dev/urandom | tr -dc 'a-z0-9' | fold -w 5 | head -n 1)
RUNNER_NAME="dockerRunner-${RUNNER_SUFFIX}"

REG_TOKEN=$(curl -sX POST -H "Accept: application/vnd.github.v3+json" -H "Authorization: token ${GH_TOKEN}" https://api.github.com/repos/${GH_OWNER}/${GH_REPOSITORY}/actions/runners/registration-token | jq .token --raw-output)

cd /home/docker/actions-runner

./config.sh --unattended --url https://github.com/${GH_OWNER}/${GH_REPOSITORY} --token ${REG_TOKEN} --name ${RUNNER_NAME}

cleanup() {
    echo "Removing runner..."
    ./config.sh remove --unattended --token ${REG_TOKEN}
}

trap 'cleanup; exit 130' INT
trap 'cleanup; exit 143' TERM

./run.sh & wait $!
```

Please note that we need three parameters to start self-hosted runner:

1. GH_OWNER - name of the GitHub account/organization
2. GH_REPOSITORY - name of the GitHub repository
3. GH_TOKEN - token to authorize requests to GitHub API

This are the Personal Access Token scopes required:

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-08.PNG?raw=true" alt="Image not found"/>
</p>

One we have the files above ready, we can decide which Azure cloud service we want to utilize to run agent in the Docker container.

We can run the runner on our local machine to test configuration using below Docker commands:

```cmd
docker build --build-arg RUNNER_VERSION=2.294.0 --tag gh-sf-docker-runner .
docker run -e GH_TOKEN='g...' -e GH_OWNER='Daniel-Krzyczkowski' -e GH_REPOSITORY='test-sh-repo' -d gh-sf-docker-runner
```

This is example of the GitHub Actions workflow with sself-hosted runner selected to run the jobs. As we can see we can still utilize the same actions as we do on the GitHub-hosted runners:

```yml
name: Build and deploy Live Notifications Azure Function App

on:
  push:
    branches: [ main ]
    paths:
      - src/live-notifications-handler/**

  workflow_dispatch:

permissions:
      id-token: write
      contents: read
      packages: read

env:
  AZURE_FUNCAPP_NAME: func-tmf-identity-live-ntfs
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'
  AZURE_RG_NAME: rg-tmf-identity

jobs:
  build-live-ntfs-func-app:
    # Here we indicate that we want to utilize self-hosted runner:
    runs-on: self-hosted

    steps:
      - uses: actions/checkout@v2

      - name: Setup .NET version
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: '6.0.x'

      - name: Install dependencies
        run:  dotnet restore ./src/live-notifications-handler/TMF.LiveNotifications.FuncApp

      - name: Build
        run: | 
          dotnet publish ./src/live-notifications-handler/TMF.LiveNotifications.FuncApp --configuration Release --no-restore --output '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/func-app-package'
          Compress-Archive -Path '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/func-app-package/*' -DestinationPath '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/func-app-package.zip'
      - name: Test
        run: dotnet test ./src/live-notifications-handler/TMF.LiveNotifications.FuncApp --no-restore --verbosity normal

      - uses: actions/upload-artifact@v2
        with:
          name: func-app-package
          path: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/func-app-package.zip'

  deploy-live-ntfs-func-app:
    needs: [build-live-ntfs-func-app]

    runs-on: self-hosted

    steps:
      - uses: actions/download-artifact@v2
        with:
          name: func-app-package
          path: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/func-app-package'

      - name: Az CLI login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Publish Func App to Azure
        run: |
          az functionapp deployment source config-zip -g ${{ env.AZURE_RG_NAME }} -n ${{ env.AZURE_FUNCAPP_NAME }} --src '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/func-app-package.zip'
```


## Azure DevOps Self-Hosted Agents

With Azure DevOps self-hosted agents we can host our runners and customize the environment used to run jobs in Azure DevOps Pipelines. Self-hosted runners can be physical, virtual, in a container, on-premises, or in the cloud exactly like for GitHub self-hosted runners.

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-09.PNG?raw=true" alt="Image not found"/>
</p>

To run the Azure DevOps self-hosted agent we need the same two files as for GitHub self-hosted runners:

1. Dockerfile with the definition of all tools we need in the container (like Azure CLI or PowerShell) and reference to *start.sh* script to start the runner.
2. start.sh - script to run self-hosted runner which will connect to our Azure DevOps organization and project.

Here is the Dockerfile content, this will install Azure CLI together with PowerShell:

```Dockerfile
FROM ubuntu:18.04

LABEL Author="Daniel Krzyczkowski"
LABEL GitHub="https://github.com/Daniel-Krzyczkowski"
LABEL BaseImage="ubuntu:18.04"

# To make it easier for build and release pipelines to run apt-get,
# configure apt to not require confirmation (assume the -y argument by default)
ENV DEBIAN_FRONTEND=noninteractive
RUN echo "APT::Get::Assume-Yes \"true\";" > /etc/apt/apt.conf.d/90assumeyes

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    jq \
    git \
    iputils-ping \
    libcurl4 \
    libicu60 \
    libunwind8 \
    netcat \
    libssl1.0 \
  && rm -rf /var/lib/apt/lists/*

RUN curl -LsS https://aka.ms/InstallAzureCLIDeb | bash \
  && rm -rf /var/lib/apt/lists/*


ARG PS_VERSION=7.1.4
ARG PS_PACKAGE=powershell_${PS_VERSION}-1.ubuntu.18.04_amd64.deb
ARG PS_PACKAGE_URL=https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/${PS_PACKAGE}
#https://github.com/PowerShell/PowerShell/releases/download/v7.1.4/powershell_7.1.4-1.ubuntu.20.04_amd64.deb
#https://github.com/PowerShell/PowerShell/releases/download/v7.1.4/powershell-lts_7.1.4-1.ubuntu.20.04_amd64.deb
# Define ENVs for Localization/Globalization
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    # set a fixed location for the Module analysis cache
    PSModuleAnalysisCachePath=/var/cache/microsoft/powershell/PSModuleAnalysisCache/ModuleAnalysisCache \
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-Ubuntu-18.04


# Install dependencies and clean up
RUN apt-get clean

RUN apt-get update \
    && apt-get install --no-install-recommends -y \
    # curl is required to grab the Linux package
        curl \
    # less is required for help in powershell
        less \
    # requied to setup the locale
        locales \
    # required for SSL
        ca-certificates \
        gss-ntlmssp \
    # PowerShell remoting over SSH dependencies
        openssh-client \
    # Download the Linux package and save it
    && echo ${PS_PACKAGE_URL} \
    && curl -sSL ${PS_PACKAGE_URL} -o /tmp/powershell.deb \
    && apt-get install --no-install-recommends -y /tmp/powershell.deb \
    && apt-get dist-upgrade -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && locale-gen $LANG && update-locale \
    # remove powershell package
    && rm /tmp/powershell.deb \
    # intialize powershell module cache
    # and disable telemetry
    && export POWERSHELL_TELEMETRY_OPTOUT=1 \
    && pwsh \
        -NoLogo \
        -NoProfile \
        -Command " \
          \$ErrorActionPreference = 'Stop' ; \
          \$ProgressPreference = 'SilentlyContinue' ; \
          while(!(Test-Path -Path \$env:PSModuleAnalysisCachePath)) {  \
            Write-Host "'Waiting for $env:PSModuleAnalysisCachePath'" ; \
            Start-Sleep -Seconds 6 ; \
          }"


RUN pwsh -Command Install-Module AZ -Force  

# Can be 'linux-x64', 'linux-arm64', 'linux-arm', 'rhel.6-x64'.
ENV TARGETARCH=linux-x64

WORKDIR /azp

COPY ./start.sh .
RUN chmod +x start.sh

ENTRYPOINT ["./start.sh"]
```

Here is the start.sh script file:

```sh
#!/bin/bash
set -e

if [ -z "$AZP_URL" ]; then
  echo 1>&2 "error: missing AZP_URL environment variable"
  exit 1
fi

if [ -z "$AZP_TOKEN_FILE" ]; then
  if [ -z "$AZP_TOKEN" ]; then
    echo 1>&2 "error: missing AZP_TOKEN environment variable"
    exit 1
  fi

  AZP_TOKEN_FILE=/azp/.token
  echo -n $AZP_TOKEN > "$AZP_TOKEN_FILE"
fi

unset AZP_TOKEN

if [ -n "$AZP_WORK" ]; then
  mkdir -p "$AZP_WORK"
fi

export AGENT_ALLOW_RUNASROOT="1"

cleanup() {
  if [ -e config.sh ]; then
    print_header "Cleanup. Removing Azure Pipelines agent..."

    # If the agent has some running jobs, the configuration removal process will fail.
    # So, give it some time to finish the job.
    while true; do
      ./config.sh remove --unattended --auth PAT --token $(cat "$AZP_TOKEN_FILE") && break

      echo "Retrying in 30 seconds..."
      sleep 30
    done
  fi
}

print_header() {
  lightcyan='\033[1;36m'
  nocolor='\033[0m'
  echo -e "${lightcyan}$1${nocolor}"
}

# Let the agent ignore the token env variables
export VSO_AGENT_IGNORE=AZP_TOKEN,AZP_TOKEN_FILE

print_header "1. Determining matching Azure Pipelines agent..."

AZP_AGENT_PACKAGES=$(curl -LsS \
    -u user:$(cat "$AZP_TOKEN_FILE") \
    -H 'Accept:application/json;' \
    "$AZP_URL/_apis/distributedtask/packages/agent?platform=$TARGETARCH&top=1")

AZP_AGENT_PACKAGE_LATEST_URL=$(echo "$AZP_AGENT_PACKAGES" | jq -r '.value[0].downloadUrl')

if [ -z "$AZP_AGENT_PACKAGE_LATEST_URL" -o "$AZP_AGENT_PACKAGE_LATEST_URL" == "null" ]; then
  echo 1>&2 "error: could not determine a matching Azure Pipelines agent"
  echo 1>&2 "check that account '$AZP_URL' is correct and the token is valid for that account"
  exit 1
fi

print_header "2. Downloading and extracting Azure Pipelines agent..."

curl -LsS $AZP_AGENT_PACKAGE_LATEST_URL | tar -xz & wait $!

source ./env.sh

print_header "3. Configuring Azure Pipelines agent..."

./config.sh --unattended \
  --agent "${AZP_AGENT_NAME:-$(hostname)}" \
  --url "$AZP_URL" \
  --auth PAT \
  --token $(cat "$AZP_TOKEN_FILE") \
  --pool "${AZP_POOL:-Default}" \
  --work "${AZP_WORK:-_work}" \
  --replace \
  --acceptTeeEula & wait $!

print_header "4. Running Azure Pipelines agent..."

trap 'cleanup; exit 0' EXIT
trap 'cleanup; exit 130' INT
trap 'cleanup; exit 143' TERM

chmod +x ./run-docker.sh

# To be aware of TERM and INT signals call run.sh
# Running it with the --once flag at the end will shut down the agent after the build is executed
# https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-linux?view=azure-devops#run-once
./run-docker.sh "$@" & wait $!
```

Please note that we need three parameters to start self-hosted agent:

1. AZP_TOKEN - personal access token from Azure DevOps
2. AZP_URL - the url of the Azure DevOps organization (https://dev.azure.com/techmindfactory in my case)
3. AZP_AGENT_NAME - name of the agent
4. AZP_POOL - name of the agent pool in Azure DevOps

This are the Personal Access Token scopes required:

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-10.PNG?raw=true" alt="Image not found"/>
</p>

One we have the files above ready, we can decide which Azure cloud service we want to utilize to run agent in the Docker container.

We can run the agent on our local machine to test configuration using below Docker command:

```cmd
docker run -e AZP_URL=https://dev.azure.com/xxxx -e AZP_TOKEN=e... -e AZP_AGENT_NAME=selfhostedlinuxagent -e AZP_POOL=Self-Hosted-Docker azdevops-sf-docker-agent:latest
```

This is example of the Azure DevOps pipeline with self-hosted agent selected to run the jobs. As we can see we can still utilize the same tasks as we do on the Azure DevOps hosted agents:

```yml
trigger:
- master
# Here we indicate that we want to utilize self-hosted runner:
pool: Self-Hosted-Docker
steps:
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: 'npm install'
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: 'npm run build.azure'
- task: CopyFiles@2
  inputs:
    SourceFolder: '$(System.DefaultWorkingDirectory)/build'
    Contents: '**'
    TargetFolder: '$(Build.ArtifactStagingDirectory)'
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'azure-app'
    publishLocation: 'Container'
```

## Sample DevSecOps solution reference

Here is the sample solution architecture of DevSecOps on Azure I created:

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-12.png?raw=true" alt="Image not found"/>
</p>

I utilize Azure Container Apps to run Azure DevOps self-hosted agents. 

<p align="center">
<img src="/images/devisland/article94/assets/devsecopsazure-deploy-to-vnet-11.PNG?raw=true" alt="Image not found"/>
</p>


# Summary

In this article, I explained why Azure Virtual Network is important when it comes to Azure solutions security, and what are the possible ways to deploy from Azure DevOps, and GitHub to Azure resources isolated with Azure Virtual Network. It is also important to remember that there are multiple hosting options in the Azure cloud for self-hosted runners and agents, like Azure Virtual Machines or Azure Container Apps. In the next article, we will talk about how to control access to Azure resources with Azure AD and Azure RBAC.


*If you like my articles, and find them helpful, please buy me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
