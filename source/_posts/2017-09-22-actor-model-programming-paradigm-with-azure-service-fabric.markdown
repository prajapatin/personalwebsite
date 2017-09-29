---
layout: post
title:  "Actor Model based programming paradigm with Azure Service Fabric (ASF)"
description: ""
date:   2017-09-22 22:00:00
comments: true
image: /images/actormodel.png
categories: [Microservices, Azure Service Fabric, Design Patterns]
keywords: "microservices, cloud, azure, service fabric, actor model"
---
<h3>What is the Actor Model programming paradigm?</h3>

When we talk about scalability of any software systems - applications, we look at it with two aspects: (1) Whether the system is built with distributed architecture pattern (2) Are we using full capability of the hardware, where the system is hosted?

What normally people do when scalability requirement comes is that they go ahead and put multiple servers and host the application with good load balancing rules, so that immediate requirement will be satisfied. The poorly designed distributed software systems will eat-up more hardware than actually needed because the software system would not have been designed to take advantages of single server/node. If we utilize one server with it's fullest capabilities, you will need fewer number of servers to satisfy higher demand. The computer processors have certain limit and modern servers having multiple CPU cores need to be managed well in terms of software design. So to take advantage of multiple cores, we design software with multi-threading concepts to run our code concurrently and we know that how difficult is to debug and fix bugs in multi-threading applications.

Considering concepts from above paragraph, let me explain Actor Model paradigm where smallest of work is being done by an Actor (it uses messages to carry out actions on system or on other actors). The actor model is a conceptual programming model for concurrent execution with specific rules like 

  1. One unit of work is done by one actor 
  2. Each actor must have mail-box on which other actors or system can send messages 
  3. Mechanism to restore the state of an actor in case of a failure
  4. Actors will act on messages sequentially 

<h4>Actor</h4>

The Actor acts on some specific assigned work through messages it receives. In case of a failure, other actors will not be affected, the actor will have it's own state and some kind of persistent state store so that actor can restart anytime and start it's work from where it left. The actor will never share memory or state with other actors and can communicate with other actors or system only through messages. Once actor completes its work and no more messages available to work upon, actor must go in some kind of hibernation and not use system resources until new message arrives.

<h4>Mailbox</h4>

The mailbox is where the actor will receive messages in sequential order and actor will act upon it in sequential order so at a time actor will do only certain unit of work/computation. If concurrent work is needed then you need multiple actors to act upon same type of messages because one actor will only work on one message.

<h4>Implementation options</h4>

There are many options to implement Actor Model in distributed software architecture, the Erlang is the language built using the concept of actor model. There is an [Akka][] framework in Java and .Net port of the same framework as an [Akka.net][], which can be used to develop enterprise scale microservices applications. There is a cloud based microservices platform called Azure Service Fabric, about which I have written in [one of my][servicefabric] previous articles. I am going to explain very simple Actor Model based microservices application using Azure Service Fabric.

<h3>Actor Model Service using Azure Service Fabric</h3>

The source code for the sample application can be cloned/downloaded from [here][github]. The application is about setting temperature value through multiple sensor actors and temperature aggregator will collect those values and creates average temperature value. There is no need to spawn cloud cluster for Azure Service Fabric to run this sample application, the local service fabric cluster manager is enough to run and test the application. The information about Azure Service Fabric SDK and local custer manager is available [here][servicefabric].

<h4>Sensor Actor</h4>

The sensor actor has four methods: (1) To Set the temperature (2) To get the temperature (the aggregator actor will call this method to collect the temperature value) (3) To set the index value for each instance of an actor (4) To get the index value for each instance of an actor. We would be using these methods in test project, which shows how to connect with actor and call specific method. The Service Fabric template simplifies the development of Actor by hiding Message sending/receiving functionality (you can look for auto generated ActorEventSource.cs file in each actor project to understand how the message communication logic is implemented) so that we can concentrate

Below points fairly explains what Azure Service Fabric is.
  
  1. It is a Platform as a Service cloud platform for Microservices based products/applications.
  2. Prebuild programming model to develop application with Stateful, Stateless, Containerized                       microservices.
  3. Service Fabric cluster can run on any platform (WINDOWS/Linux) regardless of Azure/On-Premise/Other             Clouds.
  4. Service Fabric can deploy .NET, ASP.NET Core, node.js, Windows containers, Linux containers, Java virtual       machines, scripts, Angular, or literally anything that makes up your application.
  5. It provides development machine cluster manager locally so what you see running locally will run same way       on cloud. 
  6. Deploy different versions of the same application side by side, and upgrade each application                    independently.
  7. You can manage the lifecycle of your applications without any downtime, including breaking and nonbreaking      upgrades.
  8. High density hosting - Applications are separate from VMs and service fabric manages application. It is         possible to deploy a large number of applications to a small number of VMs.
  9. Scalability can be controlled based on your application load.

<h3>The SignalR/.Net based notification application hosted on Service Fabric</h3>

You can download the source code for the application from [here][github],  you will have to replace following placeholders with your azure resources. 

  1. "[Your Service Bus Connection String]" - To create your service bus namespace on azure, please go through       [this][servicebus] article, we will be creating topics, subscriptions for this sample application.
  2. "[signalrhost-azure-url-without-http]:[signalr-host-port]" - The signalrhost-azure-url-without-http is the      service fabric cluster host and port you can hard code as a 8000.
  3. "[Your Service Fabric Connection Endpoint]" - You will need this to deploy your microservices on service        fabric platform, it is also a service fabric cluster host without http prefixed. 

In addition to above changes, you will have to create service bus topics & subscription on Azure Service Bus as below.

  1. AbilityNotificationsBackplane - This topic would be used by SignalR for scalability of notification host.       With this service bus topic, you can scale your signalr host on cloud without worrying about from which node    your notifications will be served.
  2. AbilityNotifications - This topic will be used to send/receive notification messages.
  3. AbilityNotificationsSubscriber - This subscription is needed to route the messages received to SignalR so       that it can push those messages to web client. 


You will also need [Service Fabric SDK][servicefabric] to run the sample, the same link provides instruction on installing an sdk on Visual Studio 2015 and Visual Studio 2017. I have used Visual Studio 2017 Community Edition (it is a free and full featured visual studio, which you can download from microsoft website).

Now we are ready to understand the sample microservices application. I will try to provide some of the code references in article but if you need the full source code, you can download from github URL provided above. In the sample application, we have three microservices project and one service fabric project. The SignalRHost project is a stateless microservice and provides hosting of signalr, the NotificationDispatcher project is sending messages through service bus at 15 seconds of interval and it is a stateless microservice, The NotificationListener project is a web application where we have hosted our web page to see notification messages pushed by signalr host and it is also a stateless microservice. These all microservices are stateless because we are maitaining our message state through service bus but Service Fabric provides template to create statefull services as well. There is a fourth project - SignalROnServiceFabric, which is a service fabric project hosting all above mentioned microservices.

<h4>The SignalRHost Project</h4>

There are two code snippets, which are important to understand in this project. The first code snippet is about allowing any browser to access the host by allowing cross origin request for all domains.

{% highlight C# %}

 private static void ConfigureCors(IAppBuilder app)
 {
    app.UseCors(CorsOptions.AllowAll);
 }

{% endhighlight %}

The second snippet is about creating backplane for SignalR, where we are using service bus so that we can horizontally scale our signalr host. Also we can listen for any new messages on service bus subscription and send those messages to connected client through websocket.

{% highlight C# %}

private static void ConfigureSignalR(IAppBuilder app)
{
    app.UseAesDataProtectorProvider(SignalRHostConfiguration.EncryptionPassword);

    if (SignalRHostConfiguration.UseScaleout)
    {
        var serviceBusConfig = new ServiceBusScaleoutConfiguration(SignalRHostConfiguration.ServiceBusConnectionString, 
            SignalRHostConfiguration.ServiceBusBackplaneTopic);

        GlobalHost.DependencyResolver.UseServiceBus(serviceBusConfig);
        app.MapSignalR();
    }

    var configuration = new HubConfiguration
     { EnableDetailedErrors = true, EnableJavaScriptProxies = false };
    app.MapSignalR(configuration);

    var connectionString = SignalRHostConfiguration.ServiceBusConnectionString;
    var topicName = SignalRHostConfiguration.ServiceBusNotificationTopic;
    var topicSubscriptionName = SignalRHostConfiguration.TopicSubscriptionName;

    var client = 
    SubscriptionClient.CreateFromConnectionString(connectionString, 
    topicName, topicSubscriptionName);

    client.OnMessage(message =>
    {
        var notificationHubContext = 
        GlobalHost.ConnectionManager.GetHubContext<AbilityNotificationHub>();
        notificationHubContext.Clients.All
        .showNotificationInPage(message.GetBody<String>());
    });
}

{% endhighlight %}

<h4>The NotificationDispatcher Project</h4>

From this project, we want to send custom messages to service bus topic so that above mentioned SignalR host service can listen to those messages. The stateless service provides us entry point to run our service and we will be using it as below to send custom message every 15 seconds.

{% highlight C# %}

protected override async Task RunAsync(CancellationToken cancellationToken)
{
    
    long iterations = 0;

    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        ServiceEventSource.Current.ServiceMessage(this.Context, "Working-{0}", ++iterations);

        var connectionString = DispatcherConfiguration.ServiceBusConnectionString;
        var topicName = DispatcherConfiguration.ServiceBusNotificationTopic;

        var client = TopicClient.CreateFromConnectionString(connectionString, topicName);
        var dtTime = DateTime.Now.ToString("MM/dd/yyyy HH:mm:ss.fff", CultureInfo.InvariantCulture);
        var message = new BrokeredMessage("This is a test message generated on " + dtTime);
        client.Send(message);

        await Task.Delay(TimeSpan.FromSeconds(15), cancellationToken);
    }
}
{% endhighlight %}

<h4>The NotificationListener - a web Project</h4>

In this project, signalr javascript client is used to connect with signalr host and as soon as message is received, we are appending those messages in web page. Following is the code snippet where we are connecting with the host and reading the received messages.

{% highlight javascript %}
$(function () {
    var connection = $.hubConnection("http://" + signalRHost);
    var abilityNotificationHubProxy = connection.createHubProxy('abilityNotificationHub');
    abilityNotificationHubProxy.on('showNotificationInPage', function (message) {
        $("#notificationArea").append(message).append("<br/>");
    });
    connection.start().done(function () {
        // send notification to the server.
    });

});
{% endhighlight %}

This is the end of a small microservices based application, which is created to just get feel of what the Azure Service Fabric is. I am planning to write next blog about hosting docker container in Service Fabric and I am planning to create windows service and containerize it.

[servicefabric]: /blog/2017/signalr-based-app-on-service-fabric/
[github]: https://github.com/prajapatin/ActorPatternOnServiceFabric
