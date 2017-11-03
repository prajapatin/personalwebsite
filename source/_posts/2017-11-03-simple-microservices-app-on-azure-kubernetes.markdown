---
layout: post
title:  "Simple Microservices application on Azure Kubernetes cluster"
description: "It is a sample microservices application hosted on kubernetes cluster on Azure platform."
date:   2017-11-03 22:00:00
comments: true
image: /images/microservices-kubernetes.png
categories: [Cloud, Microservices, Kubernetes]
keywords: "Kubernetes, microservices, cloud, azure"
---
<h3>Background of sample application & technologies/tools used for the development</h3>

Previously I have written [introductory post][microservicesintro] about microservices application and also written about Azure Service Fabric based microservices application [here][signalrpost] and [here][actormodelpost]. What I would like to write about in this post is developing microservices app without using any PaaS based microservices platform. The content is going to be little longer so you may have to spend some extra time reading this post and the reason is that I wanted to write about each and every step for the development and provide resource reference for each step/topic. If you have gone through above mentioned introductory post, you would have noticed that there are multiple characteristics of microservices and we may not cover each and every characteristic in our sample application. 

We are going to create two micro services making one application and host each micro service in docker container. The container service we have chosen is Azure Container Service and [Kubernetes][kuberneteslink] as a orchestration platform. The microservices application might need multiple version upgrade in a day/week and each service has to be updated/versioned independently, scaled up/down independently and orchestration platform provides all these functionalities. The Kubernetes cluster can be deployed on AWS as well without making any changes. Following is the detail about each micro service and we have chosen NodeJS as a frontend service and .Net Core (on Linux) as a back-end service, that is the beauty of containers (you can mix any technology components to make your application). I have explained in one of my [previous][windowsservicepost] articles about how to install docker on Windows machine, we can build both Linux and windows based images using docker. For this sample application, we are using Linux based images. I will add one more micro service written using Java when I get some time but for now we will stick with two services. You can download the source code from [here][github] so that you can play with it while I describe each part, you will need Visual Studio 2017(Community version or any other editions).

<h3>Messaging Client - Frontend</h3>

This is a simple NodeJS-ExpressJs frontend application, where we are showing notification messages. We are using Socket-io library to push notification from server to browser. The NodeJS server is getting messages from Azure Service Bus topic and pushing those messages to client/browser using Socket-io library. We need to create docker image from this application and push it to public repository so that when we deploy kubernetes, it can pull image from the repository. Below I have explained some NodeJS code and docker file and also how to build docker image & push to the server. 

{% highlight JavaScript %}

var io = require('socket.io')(server);
var allConnections = [];
var timerInstance;
io.sockets.on('connection', function (socket) {
    allConnections.push(socket);
    if(!timerInstance) {
        var receiveMessage = function () {
            try {
                serviceBusService.receiveSubscriptionMessage(process.env.topicName, process.env.subscriptionName, function (error, receivedMessage) {
                    if (!error) {
                        for(var connection in allConnections){
                            allConnections[connection].emit('receiveMessage', { message: receivedMessage.body });
                        }
                        
                    }
                });
            } catch (e) {
            }
            timerInstance = setTimeout(receiveMessage, 2000);
        };
        receiveMessage();
    }
    socket.on('disconnect', function() {
        var socketIndex = allConnections.indexOf(socket);
        if(allConnections.length > socketIndex){
            allConnections.splice(socketIndex, 1);
        }
     });
});

{% endhighlight %}

I am highlighting above code in the post because here we are using two interesting libraries. The first one is azure npm library, it is JavaScript library to work with many Microsoft Azure services. We are using azure library to subscribe to Azure Service Bus topic, from where we would be able to receive messages. Currently there is no way through this library to continuously listen to topic so I have added timer and it is not recommended way for production. I am concentrating here more on explaining microservices but if you really need an AMQP based library then you should look for library like [this][amqp10] for production. The other library is Socket-io, using which we are pushing messages to all connected browsers as and when we receive message from Azure Service Bus. We are also using environment variables for different type of settings so that when we run docker image as a container, we can pass those environment variables.

Below is what we are using to build docker image for client service, you will find dockerfile in MessagingClient folder.

{% highlight PowerShell %}

FROM node:7
WORKDIR /app
COPY package.json /app
RUN npm install
COPY public /app/public
COPY routes /app/routes
COPY views /app/views
COPY app.js /app
CMD node app.js
EXPOSE 80
EXPOSE 5671

{% endhighlight %}   

The above dockerfile will take node base image from docker hub and copy required folder/files along with packages.json file, using which npm install command will pull required libraries in working directory. It also exposes required ports and CMD line will be executed when we run the container.


{% highlight PowerShell %}
$ docker build -t messagingclient .
{% endhighlight %}

The above command will build the docker image and you need to make sure that your command prompt is pointing to directory where your application and dockerfile is residing.

<h3>Messaging Api - backend</h3>

This is a simple web api project targeted to .Net Core 2.0 so that the web api code is portable and can run on linux. We are using .Net Standard API, which is a kind of wrapper API to target different variants of platform. In some scenarios, we would not know whether certain APIs will work on Mono or specific variant of Xamarin  so .Net standard APIs simplifies those scenarios and if API is available in Standard interface then you are sure that your app will work across. In this project, we are building docker image differently. We are using docker compose and Visual Studio has a docker compose project to build the docker image from the C# project. We are exposing messages API from this project, which will accept the message and push to the Azure Service Bus topic. The NodeJS client will call this API to post the message but you can utilize tool like [Postman][getpostman] to post the message, which will be delivered back to NodeJS client using Azure Service Bus topic subscription.

The dockerfile is available in project, which would be used by docker compose to build the image.

{% highlight PowerShell %}

FROM microsoft/aspnetcore:2.0
ARG source
WORKDIR /app
ENV ASPNETCORE_URLS http://+:3000
EXPOSE 3000
EXPOSE 5671
COPY ${source:-obj/Docker/publish} .
ENTRYPOINT ["dotnet", "MessagingAPI.dll"]

{% endhighlight %}

The above dockerfile instructs docker runtime to pull asp.net core portable image from docker hub and provide the entry point to host/run Web API project. 

<h3>Hosting microservices using Kubernetes cluster on Microsoft Azure</h3>

I would like to explain more about YAML config file to deploy the Kubernetes services instead of explaining about Azure Kubernetes Service. If you want to quickly read about Azure Kubernetes Service, I would recommend [this][aksintro] article. The article explains about installing aks CLI and also you will need kubectl CLI to work with deployed kubernetes cluster. To use docker images we created above, we will have to push those images to docker hub so that kubernetes can pull those images.

<h4>How to push docker images from local machine to docker hub?</h4>

You will have to login to docker using following command.

{% highlight PowerShell %}
docker login
{% endhighlight %}

The command prompt will ask you for the docker credentials, which you can provide and then look at below commands.

{% highlight PowerShell %}
docker tag {NameofYourLocalImage} {TagForYourLocalImage}
docker push {TagForYourLocalImage}
{% endhighlight %}

The first command from above snippet will tag your local image and create tagged image from it which we are pushing it to docker hub into your account. You will have to replace name of your image with curly brackets. We would be using these public docker repositories, we created into our YAML config to deploy kubernetes cluster. As I said previously, you will have to make sure that AKS is installed and you have followed all the commands mentioned in [Microsoft article][aksintro] I shared so that Kubernetes cluster is ready to use for deployment. I will first introduce you to basic kubectl commands and then we can go into detail of YAML config.

<h4>Understanding kubernetes deployment</h4>

{% highlight PowerShell %}

kubectl get node
kubectl create -f {PathToYAMLConfigFile}
kubectl get service messeging-api --watch
kubectl get service messeging-client --watch
kubectl describe deployment

{% endhighlight %}

The first command from above snippet will retrieve the number of agent/node running in cluster and second command will actually deploy services as per detail provided in YAML file. The next two commands will help you to retrieve external IP of each service so that you can browse through it. The last command will give you complete deployment detail of full cluster with all services. There are other commands which are useful but you can browse through [here][kuberneteslink] to know about all capabilities of Kubernetes Orchestration platform. Below I have explained different portion of YAML configuration, available in aks-app.yml file for kubernetes deployment.

{% highlight javascript %}
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: messeging-api
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: messeging-api
        tier: backend
        track: stable
    spec:
      containers:
      - name: messeging-api
        image: nileshprajapati/messagingapi
        ports:
        - containerPort: 3000
          name: messagingapi
        - containerPort: 5671
          name: amqp
        env:
        - name: serviceBusConnectionString
          value: "{Your Service Bus connection string}"
        - name: serviceBusEntityPath
          value: "microservicesmessages"
---
apiVersion: v1
kind: Service
metadata:
  name: messeging-api
spec:
  selector:
    app: messeging-api
    tier: backend
  type: LoadBalancer
  ports:
  - port: 3000
    name: messagingapi
  - port: 5671
    name: amqp
  selector:
    app: messeging-api
---
{% endhighlight %}

The first section from above config, defines the kubernetes pod. A pod is a group of one or more containers, with shared storage/network, and a specification for how to run the containers. A pod’s contents are always co-located and co-scheduled, and run in a shared context. We have only one container so that we have only one container for our back-end and the image location is nileshprajapati/messagingapi, which is available as a public image in docker hub. If you have a private repository then it will need credential to pull images from it, which is I am ignoring to explain. You will also notice that there is a env key, that is where we provide all environment variable values. These are the environment variables used by our service and different values can be supplied here for different types of deployments. It also has port information where you will map your container port with target machine/node where your service would be running.

The second section defines the service where you can specify load balancer as well. When we specify type as load balancer, the kubernetes will use default load balancer provided by cloud vendor where you would be running the cluster. The important key we need to keep in mind is app under selector key, which we would be using to connect our frontend service to it. This was our back-end service, now we will look at the frontend service config.

{% highlight javascript %}

apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: messeging-client
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: messeging-client
        tier: frontend
        track: stable
    spec:
      containers:
      - name: messeging-client
        image: nileshprajapati/messagingclient
        ports:
        - containerPort: 80
          name: messegingclient
        - containerPort: 5671
          name: amqp
        env:
        - name: serviceBusConnectionString
          value: "{Your Service Bus connection string}"
        - name: topicName
          value: "microservicesmessages"
        - name: subscriptionName
          value: "messageconsumer"
        - name: messagingAPI
          value: "messeging-api"       
---
apiVersion: v1
kind: Service
metadata:
  name: messeging-client
spec:
  type: LoadBalancer
  ports:
  - port: 80
    name: messegingclient
  - port: 5671
    name: amqp
  selector:
    app: messeging-client
  sessionAffinity: ClientIP

{% endhighlight %}

The explanation remains the same for client service as well with couple of points to understand. The first one is that we are setting replicas as a 1 here because we want to run only one instance (one container). Also you will notice that in environment variables, we are giving one value "messeging-api" to messagingAPI key, which is actually our back-send service. You would be wondering then that how our client will connect with service using that simple name as it is not even DNS name. The magic here is that kubernetes will try to connect locally using selector we have specified in back-end service and resolve the call to that service running within a cluster. If we do not get this functionality from kubernetes then you will have to actually supply external IP there and you know that which is not possible in dynamic cloud deployment scenarios. There are few commands to scale-up and scale-down individual service in cluster, which will create/delete additional replicas of the specific service.

This is a brief introduction about simple microservices app hosted in kubernetes cluster and I know that I could not explain each and every detail but please connect with me if you have any specific queries on any of the service or kubernetes.  


[microservicesintro]: /blog/2017/what-is-micro-services-architecture/
[signalrpost]: /blog/2017/signalr-based-app-on-service-fabric/
[actormodelpost]: /blog/2017/actor-model-programming-paradigm-with-azure-service-fabric/
[windowsservicepost]: /blog/2017/windows-service-as-a-container-on-service-fabric/
[getpostman]: https://www.getpostman.com/
[kuberneteslink]: https://kubernetes.io/
[github]: https://github.com/prajapatin/SampleMicroservicesApp
[amqp10]: https://www.npmjs.com/package/amqp10
[aksintro]: https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough