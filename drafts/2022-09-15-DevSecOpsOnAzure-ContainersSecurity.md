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

This is the next article from the series called *DevSecOps practices for Azure cloud workloads*. In this article I would like to talk about containers security in the Azure cloud, and how to enhance it.

# Apply Docker images security best practices

Utilizing Docker containers has many advantages, however as with many other solutions, there are security threats. This is why when building container solutions on Microsoft Azure cloud, it is important to keep an eye on the containers security. In this section I would like to go through some Docker images security best practices.

## Create a dedicated user on the image (avoid root)

Processes in a container should not run as root. Instead, dedicated user should be created in the Dockerfile, and process should be run as this user. Images that follow this pattern are easier to run securely by limiting access to resources. Here is example:

```YAML
FROM base-image:latest
RUN apt install ...
USER docker-user:demo-user-group
ENTRYPOINT ["demo-binary"]
```

Containers started from this image will run as docker-user. The user will be a member of the *demo-user-group*. You can omit the group name if you don’t need the user to be in a group by using just *USER docker-user*.

## Find, fix and monitor for open source vulnerabilities

Scan Docker images for known vulnerabilities and integrate it as part of Continuous Integration. Before pushing Docker images to container registry, there are many tools that can be used on the development machine to can Docker images and detect security issues. In this article I would like to present how to utilize [*Docker Scan tool*](https://docs.docker.com/engine/scan/) powered by [Snyk](https://snyk.io/) which is included in the latest Docker Desktop. Vulnerability scanning for Docker local images allows developers and development teams to review the security state of the container images and take actions to fix issues identified during the scan, resulting in more secure deployments. Docker Scan runs on Snyk engine, providing users with visibility into the security posture of their local Dockerfiles and local images.

It is worth to mention that Docker Scan provides 10 free scans per month for authenticated users in to Docker Desktop. If we need more, we can buy [Snyk paid plan](https://snyk.io/plans/) with containers scanning capability. With free plan we get 100 Container tests per month.

How can we scan local Docker images? It is easy, below I provided few commands:

### Run a single test on tagged image:

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

Unfortunately sometimes we can accidentally leak secrets, tokens, and keys into Docker images when we build them. To reduce the risk it is recommended to use multi-stage Docker builds, and a *.dockerignore* file to avoid COPY instruction, which pulls in sensitive files that are part of the build contex.

### Use fixed tags for immutability

Docker image owners can push new versions to the same tags. This can result in inconsistent images during builds, and makes it hard to track fixes for vulnerabilities. To make it easier, we can add image hash, like: *FROM mcr.microsoft.com/dotnet/aspnet:<hash>*.

### Use COPY instead of ADD

In the [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy) we can find information that *COPY* command is preferred. This is because *COPY* does not support *src* with URL scheme, and does not unpack compression files (like TAR). *COPY* lets us to copy in a local file or directory only from our host (the machine building the Docker image) into the Docker image itself. *ADD* lets us do exactly the same, but it also supports 2 other sources. We can use a URL instead of a local file/directory. Secondly, we can extract a tar file from the source directly into the destination which can lead to security issues. When remote URLs are used to download data directly into a source location, they could result in man-in-the-middle attacks that modify the content of the file being downloaded. Moreover, the origin and authenticity of remote URLs need to be further validated. Downloading TAR/ZIP files adds the risk of [zip bombs attacks](https://en.wikipedia.org/wiki/Zip_bomb).

### Use a linter

We can enforce Dockerfile best practices automatically by using a static code analysis tool such as [hadolint linter](https://github.com/hadolint/hadolint), that will detect and alert for issues found in a Dockerfile:

<p align="center">
<img src="/images/devisland/article91/assets/devsecopsazure-azure-containers-sec05.png?raw=true" alt="Image not found"/>
</p>


# Securely store Docker images in the Azure cloud

Aaa

# Enhance security with Microsoft Defender for Containers

Aaa

# Summary

In this article, I explained ...


*If you like my articles, and find them helpful, please buy me a coffee:*

<a href="https://www.buymeacoffee.com/techmindfactory" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/arial-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>