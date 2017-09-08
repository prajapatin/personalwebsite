---
layout: post
title:  "Windows Service as a Container on Azure Service Fabric"
description: "A dockerized Windows Service container hosted as a microservice on Azure Service Fabric"
date:   2017-09-05 21:00:00
comments: true
image: /images/docker.png
categories: [Cloud, Microservices, Docker Container]
keywords: "microservices, cloud, azure, service fabric, docker container, windows service"
---
<h3>What is Docker and Container?</h3>

The [Docker][dockerlink] is a container platform using which you can create software containers. The container is a packaged software which shares the host operation system and contains all dependencies (System files, third-party software, settings or anything which is needed to run your containerized app) needed to run your software application.

Containers are used to ship software faster and developers need not to be worried about whether the containerized app will run on specific machine or not. They are very light weight and share host operating system where you are running them and you can run multiple sandboxed containerized apps on specific VM or physical machine. There are many software product companies around the world which are using containers to release software on daily basis and saying multiple times in a day is also not wrong. 

We are going to use Docker for Windows for this demo application and you will need to install [Docker for Windows][dockerforwindows] on Windows 10 (Anniversary edition) or Windows Server 2016. Also you will have to enable hardware virtualization through BIOS to install Docker runtime. 

<h3>What is our sample app?</h3>

I have chosen windows service as an app to containerize because there are not many online references on containerizing custom windows service app in container. The source code is available [here][githublink] and it contains very simple windows service hosting OWIN based web api (RESTful service). It also contains Service Fabric project using which you can host container in Azure Cloud as a microservice. You will not be able to host containers in local service fabric cluster unless you have a Windows Server 2016 so you will have to spawn Service Fabric Cluster on Azure if you would want to test it on cloud. If you are not going to host container in cloud then I am going to show you how you can create Docker image and run locally.

<h3>Containerizing and running sample app</h3>

As a first step, make sure that Docker is properly installed on your machine and Docker runtime is up and running. Now once you take the sample code from above mentioned github repository, build the solution. You will notice dockerfile (Docker template file to build the Docker image) & init.ps1(PowerShell script to be executed) files in release/debug within bin folder based on build configuration you have chosen, we are going to use these two files for building and running our Docker container.

First we will try to understand dockerfile template.

{% highlight PowerShell %}

FROM microsoft/dotnet-framework:4.7
RUN mkdir C:\installation
ADD . /installation
RUN sc create WebAPIHostService start=auto binpath="C:\installation\HostService.exe"
CMD powershell C:\installation\init.ps1
EXPOSE 9000

{% endhighlight %}

To run the Docker build command, you will need to open PowerShell command prompt in administartor mode and then set your directory to bin/debug(or release) folder of the windows service project you have built as per above instructions. The above Docker template says that create the Docker image by taking microsoft/dotnet-framework:4.7 as a base image and create the installation folder in c directory in Docker image. The third line says that add whatever current directory content to created folder. In the fourth line we are instructing Docker runtime to create windows service from provided exe. The line prefixed with CMD instructs Docker runtime to run specified powershell script when we would be running Docker container from Docker image. The last line exposes port 9000, on which we can access REST api hosted in windows service.

Now you have understood dockerfile, we will run following command in PowerShell, which will create Docker image.

{% highlight PowerShell %}
docker build -t webapiservice .
{% endhighlight %}

The command will take quite a bit of time as it will download base windows image with .Net Framework from the Docker hub, it will download the image only first time and thereafter if you build any Docker image from the mentioned base image; it will use local image from your machine. Once the image creation is successful, you can verify that your image is created using below PowerShell command.

{% highlight PowerShell %}
docker images
{% endhighlight %}

We now have a Docker image and we want to run container using that image so we need to run following command.

{% highlight PowerShell %}
docker run --name webapiservice -d -p 9000:9000 webapiservice
{% endhighlight %}

The above command runs Docker container with name as a 'webapiservice' and maps container port 9000 with host port 9000. Once you run the command, The Docker runtime will start the container and it will use PowerShell script provided in template as a CMD. Let us try to understand that init.ps1 file, here we are starting wendows service as soon as container starts. We are also making sure that we are making http call to our REST service and flushing out logs if any so that we can verify those logs running 'Docker logs {nameofcontainer}' PowerShell command.

{% highlight PowerShell %}

Write-Output 'Starting Web API Host Server'
Start-Service WebAPIHostService
    
Write-Output 'Making HTTP GET call'
Invoke-WebRequest http://localhost:9000/api/welcome -UseBasicParsing | Out-Null

Write-Output 'Flushing log file'
netsh http flush logbuffer | Out-Null

Write-Output 'Tailing log file'
Get-Content -path 'C:\installation\service.log' -Tail 1 -Wait

{% endhighlight %}

Once command runs successfully, we can verify that your container is running by below command.

{% highlight PowerShell %}
docker ps
{% endhighlight %}

If you want to verify that your windows service is running successfully, you will need to get IP address of container using below command.

{% highlight PowerShell %}
docker inspect --format="{ {.NetworkSettings.Networks.nat.IPAddress} }" webapiservice
{% endhighlight %}

Once you get an IP address of running container, you can browse through {IP Address}:9000/api/welcome and verify that your API is up and running. You can also try to browse through another method {IP Address}:9000/api/welcome/detail. We now have windows service running in container so we can quickly look at how we can run container on Azure Service Fabric.

<h3>Hosting windows service container on Azure Service Fabric</h3>

In the sample solution, you will find ServiceFabricHost project and following content explains the detail about hosting container on service fabric. My [previous article][previousarticle] explains about installing Service Fabric SDK, which is prerequisite to host this application on Azure Service Fabric.

Following portion of ServiceManifest.xml file should have an Azure Container Registry path of your container registry, about which is explained in [previous article][previousarticle]. For your image URL, you will have to use full path including your container registry URL.

{% highlight XML %}
<EntryPoint>
    <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
    <ContainerHost>
        <ImageName>[Your container image URL]</ImageName>
    </ContainerHost>
</EntryPoint>
{% endhighlight %}

Also you will need to understand following portion from the same file, where we are creating endpoint for our microservice.

{% highlight XML %}
<Resources>
    <Endpoints>
       <!-- This endpoint is used by the communication listener to obtain the port on which to 
            listen. Please note that if your service is partitioned, this port is shared with 
            replicas of different partitions that are placed in your code. -->
       <Endpoint Name="WebAPIHostServiceTypeEndpoint" Protocol="http" Port="9000" />
    </Endpoints>
</Resources>
{% endhighlight %}

The another file ApplicationMenifest.xml is also very important to understand as it explains port mapping between Container and host and credential to connect with Azure Container Registry so that Service Fabric can download the docker image. We need to know another very important detail about endpoint reference in port mapping, it is a same endpoint name we have provided in ServiceManifest.xml.

{% highlight XML %}
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="WebAPIHostServicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
            <RepositoryCredentials AccountName="[Your Container Registry Account Name]" Password="[Container Registry Account Password]" Email="[You email address associated with Azure account]" PasswordEncrypted="false"/>
            <PortBinding ContainerPort="9000" EndpointRef="WebAPIHostServiceTypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
</ServiceManifestImport>
{% endhighlight %}

So once you understand and provide your detail in configuration, you are ready to publish service fabric application.

[dockerlink]: https://www.docker.com/what-docker
[dockerforwindows]: https://store.docker.com/editions/community/docker-ce-desktop-windows
[githublink]: https://github.com/prajapatin/WindowsServiceContainerOnASF
[previousarticle]: /blog/2017/signalr-based-app-on-service-fabric/
