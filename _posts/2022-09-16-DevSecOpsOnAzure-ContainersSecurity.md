---
title: "DevSecOps on Azure - part3: Enhance containers security in Azure cloud"
excerpt: "This article presents security practices for containers in the Azure cloud"
header:
  image: /images/devisland/article91/assets/devsecopsazure-azure-containers-sec01.png
---

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec01.png?raw=true" alt="DevSecOps on Azure - part3"/>
</p>

# Introduction

This is the next article from the series called *DevSecOps practices for Azure cloud workloads*. In this article I would like to talk about container security in the Azure cloud, and how to enhance it.

# Apply Docker images security best practices

Utilizing Docker containers has many advantages, however, as with many other solutions, there are security threats. This is why when building container solutions on the Microsoft Azure cloud, it is important to keep an eye on the container's security. In this section, I would like to go through some Docker image security best practices.

### Create a dedicated user on the image (avoid root)

Processes in a container should not run as root. Instead, the dedicated user should be created in the Dockerfile, and the process should be run as this user. Images that follow this pattern are easier to run securely by limiting access to resources. Here is an example:

```YAML
FROM base-image:latest
RUN apt install ...
USER docker-user:demo-user-group
ENTRYPOINT ["demo-binary"]
```

Containers started from this image will run as docker-user. The user will be a member of the *demo-user-group*. You can omit the group name if you don’t need the user to be in a group by using just *USER docker-user*.

### Find, fix and monitor for open source vulnerabilities

Scan Docker images for known vulnerabilities and integrate them as part of Continuous Integration. Before pushing Docker images to the container registry, many tools can be used on the development machine to scan Docker images and detect security issues. In this article, I would like to present how to utilize [*Docker Scan tool*](https://docs.docker.com/engine/scan/) powered by [Snyk](https://snyk.io/) which is included in the latest Docker Desktop. Vulnerability scanning for Docker local images allows developers and development teams to review the security state of the container images and take action to fix issues identified during the scan, resulting in more secure deployments. Docker Scan runs on the Snyk engine, providing users with visibility into the security posture of their local Dockerfiles and local images.

It is worth mentioning that Docker Scan provides 10 free scans per month for authenticated users in Docker Desktop. If we need more, we can buy [Snyk paid plan](https://snyk.io/plans/) with container scanning capability. With the free plan, we get 100 Container tests per month.

How can we scan local Docker images? It is easy, below I provided a few commands:

### Run a single test on the tagged image:

```text
 docker scan tmfdevsecopsapi:dev
```

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec02.png?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec03.png?raw=true" alt="Image not found"/>
</p>

For popular official images on Docker Hub, this will provide base
image recommendations and alternate base images that can help
reduce vulnerabilities

### Use the Dockerfile to generate a more detailed analysis:

```text
docker scan tmfdevsecopsapi:dev --file C:\Users\DanielKrzyczkowski\Desktop\src\TMF.DevSecOps.API\Dockerfile
```

### Ignore any vulnerabilities from the base image:

```text
docker scan tmfdevsecopsapi:dev --exclude-base --file C:\Users\DanielKrzyczkowski\Desktop\src\TMF.DevSecOps.API\Dockerfile
```

Scanning result will only show vulnerabilities introduced by us in the Docker image:

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec04.png?raw=true" alt="Image not found"/>
</p>

We can also filter results out by using severity level: *--severity high*.

We can also utilize [Snyk CLI for container security](https://docs.snyk.io/products/snyk-container/snyk-cli-for-container-security) directly. Once we have Snyk CLI installed, we can scan our Docker images, and files:

Scan a docker image with Snyk:

```text
$ snyk test --docker tmfdevsecopsapi:dev --file=path/to/Dockerfile
```

Use Snyk to monitor and alert to newly disclosed vulnerabilities in a docker image:

```text
$ snyk monitor --docker tmfdevsecopsapi:dev
```

### Do not leak sensitive information to Docker images

Unfortunately, sometimes we can accidentally leak secrets, tokens, and keys into Docker images when we build them. To reduce the risk it is recommended to use multi-stage Docker builds, and a *.dockerignore* file to avoid COPY instruction, which pulls in sensitive files that are part of the build context.

### Use fixed tags for immutability

Docker image owners can push new versions to the same tags. This can result in inconsistent images during builds and makes it hard to track fixes for vulnerabilities. To make it easier, we can add image hash, like: *FROM mcr.microsoft.com/dotnet/aspnet:<hash>*.

### Use COPY instead of ADD

In the [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy) we can find information that *COPY* command is preferred. This is because *COPY* does not support *src* with URL scheme, and does not unpack compression files (like TAR). *COPY* lets us to copy in a local file or directory only from our host (the machine building the Docker image) into the Docker image itself. *ADD* lets us do exactly the same, but it also supports 2 other sources. We can use a URL instead of a local file/directory. Secondly, we can extract a tar file from the source directly into the destination which can lead to security issues. When remote URLs are used to download data directly into a source location, they could result in man-in-the-middle attacks that modify the content of the file being downloaded. Moreover, the origin and authenticity of remote URLs need to be further validated. Downloading TAR/ZIP files adds the risk of [zip bombs attacks](https://en.wikipedia.org/wiki/Zip_bomb).

### Use a linter

We can enforce Dockerfile best practices automatically by using a static code analysis tool such as [hadolint linter](https://github.com/hadolint/hadolint), that will detect and alert for issues found in a Dockerfile:

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec05.png?raw=true" alt="Image not found"/>
</p>


# Scan Docker images in the Continuous Integration process

Even if we do our best to eliminate potential security issues for our Docker images, there is still a chance that we did not notice something. Additionally, scanning locally is for our knowledge (developers). It would be nice to have an additional scanning gateway in our CI/CD pipeline so we can generate reports to be verified by the security team. This is why it can be helpful to integrate Docker image scanning in the Continuous Integration pipeline. Below I present how to integrate Snyk containers scanning in the Azure DevOps pipeline, and GitHub Actions.

### Snyk containers scanning in Azure DevOps

Under [this link](https://docs.snyk.io/integrations/ci-cd-integrations/azure-pipelines-integration) you can find full explanation how to integrate Snyk with Azure DevOps. Here is the task which will scan DockerFile:

```YAML
- task: SnykSecurityScan@1
  inputs:
    serviceConnectionEndpoint: 'snyk-connection'
    testType: 'container'
    dockerImageName: 'tmf-dev-sec-ops'
    dockerfilePath: 'Dockerfile'
    failOnIssues: true
    monitorWhen: 'always'
```

### Snyk containers scanning on GitHub

Under [this link](https://docs.snyk.io/integrations/ci-cd-integrations/azure-pipelines-integration) you can find a full explanation of how to integrate Snyk with Azure DevOps. Here is the task which will scan DockerFile:

```YAML
name: Docker images scanning with Snyk 
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
    - name: Run Snyk to check Docker images for vulnerabilities
      uses: snyk/actions/docker@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        image: tmf-dev-sec-ops
        args: --severity-threshold=high
```

We can also push the report to GitHub so it security report will be visible under *Security* tab:

```YAML
name: Snyk Container
on: push
jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Build a Docker image
      run: docker build -t tmf-dev-sec-ops .
    - name: Run Snyk to check Docker image for vulnerabilities
      continue-on-error: true
      uses: snyk/actions/docker@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        image: tmf-dev-sec-ops
        args: --file=Dockerfile
    - name: Upload result to GitHub Code Scanning
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: snyk.sarif
```


# Securely store Docker images in the Azure cloud

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec06.png?raw=true" alt="Image not found"/>
</p>

Once we have a stable process around scanning Docker images locally, and in the CI/CD pipeline, it is important to store these images securely. In the Azure cloud, there is a dedicated service for storing Docker images with different tags, called [Azure Container Registry (ACR)](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-intro). Thanks to Azure Container Registry features we can make sure that our Docker images will be stored, and maintained securely.

## ACR Tasks

Azure Container Registry Tasks is a suite of features within Azure Container Registry. It provides cloud-based container image building for platforms including Linux, Windows, and ARM, and can automate OS and framework patching for Docker containers. ACR Tasks support several scenarios to build and maintain container images. One of the interesting Tasks can automate OS and framework patching. This Task can track a dependency on a base image when it builds an application image. When the updated base image is pushed to our registry or a base image is updated in a public repo such as in Docker Hub, ACR Tasks can automatically build any application images based on it. With this automatic detection and rebuilding, ACR Tasks save the time and effort normally required to manually track and update every application image referencing your updated base image. I encourage you to read more about [ACR Tasks capabilities](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-tasks-overview).

# Enhance security with Microsoft Defender for Containers

With all the above we definitely can improve security around Docker containers that are running and are stored in the Azure cloud. Microsoft Defender for Containers is the cloud-native solution that is used to secure containers so we can improve, monitor, and maintain the security of our Kubernetes clusters and Docker containers.

Defender for Containers supports the three core aspects of container security:

**1. Environment hardening** - Defender for Containers protects Kubernetes clusters whether they're running on Azure Kubernetes Service, Kubernetes on-premises/IaaS, or Amazon EKS. Defender for Containers continuously assesses clusters to provide visibility into misconfigurations and guidelines to help mitigate identified threats.

**2. Vulnerability assessment** - Vulnerability assessment and management tools for images stored in ACR registries and running in Azure Kubernetes Service.

**3. Run-time threat protection for nodes and clusters** - Threat protection for clusters and Linux nodes generates security alerts for suspicious activities.

I encourage you to read more in the [official documentation](https://docs.microsoft.com/en-us/azure/defender-for-cloud/defender-for-containers-introduction).

### Scanning images stored in Azure Container Registry

Above I mentioned that we can utilize Azure Container Registry to store our Docker images securely in the Azure cloud. When Defender for Containers is enabled, it offers vulnerability scanning for images in Azure Container Registries (ACRs). Triggers for scanning are activated when an image is pushed into a registry (on push). There are also weekly scans of images that have been pulled in the last 30 days, or when we import images into an ACR.

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec07.png?raw=true" alt="Image not found"/>
</p>

By default, Defender for Containers is disabled, we can enable it for our Azure subscription:

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec08.PNG?raw=true" alt="Image not found"/>
</p>

Then we have access to recommendations directly from the ACR blade:

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec09.PNG?raw=true" alt="Image not found"/>
</p>

It is worth mentioning that Defender for Cloud can provide the recommendation, by correlating the inventory of our running containers that are collected by the Defender agent which is installed on our Azure Kubernetes Service clusters, with the vulnerability assessment scan of images that are stored in Azure Container Registry. The recommendation then shows us running containers with the vulnerabilities associated with the images that are used by each container and provides us with vulnerability reports and remediation steps.


# Summary

In this article, I explained how to improve the security of Docker images, and containers. We discovered how to scan Docker images, and files locally, to detect potential vulnerabilities. As we discovered, we can improve Docker containers' security at each stage: on the local machine, in CI/CD pipeline, in the Azure Container Registry, and on the running environment, like Kubernetes. In the next article, we will discover how in general enhance security in Continuous Integration pipelines.


*If you like my articles, and find them helpful, please buy me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
