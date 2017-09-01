---
layout: post
title: "Creating SignalR based microservices app on Azure Service Fabric"
description: "A SignalR based sample application walkthrough, which is using microservices hosted on azure service fabric"
date: 2017-08-31 19:00:00
comments: true
image: /images/signalr.png
categories: [Cloud]
keywords: "microservices, cloud, azure, service fabric, signalr"
---
<h3>What is Azure Service Fabric?</h3>

You can read about Azure Service Fabric in detail from [here.][servicefabricintro] I will try to summarize it if you do not have a time to go through it in detail. In my previous [article][microservices], I have explained about microservices so the Azure Service Fabric is a platform to develop, deploy and monitor the microservices based applications. You need to spawn Service Fabric Cluster on Azure to host and run service fabric based microservices application for production deployment otherwise development machine is enough to run and host service fabric application locally using development cluster manager. If application runs locally as per the requirement, it can be packaged and published directly to Azure Service Fabric Cluster on cloud and can be monitored using service fabric explorer (web based tool). 

Below points fairly explains what Azure Service Fabric is.

<ol>
  <li>It is a Platform as a Service cloud platform for Microservices based products/applications.</li>
  <li>Prebuild programming model to develop application with Stateful, Stateless, Containerized                       microservices.</li>
  <li>Service Fabric cluster can run on any platform (WINDOWS/Linux) regardless of Azure/On-Premise/Other             Clouds.</li>
  <li>Service Fabric can deploy .NET, ASP.NET Core, node.js, Windows containers, Linux containers, Java virtual       machines, scripts, Angular, or literally anything that makes up your application.</li>
  <li>It provides development machine cluster manager locally so what you see running locally will run same way       on cloud.</li> 
  <li>Deploy different versions of the same application side by side, and upgrade each application                    independently.</li>
  <li>You can manage the lifecycle of your applications without any downtime, including breaking and nonbreaking      upgrades.</li>
  <li>High density hosting - Applications are separate from VMs and service fabric manages application. It is         possible to deploy a large number of application to a small number of VMs.</li>
  <li>Scalability can be controlled based on your application load.</li>
</ol>

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

    var configuration = new HubConfiguration { EnableDetailedErrors = true, EnableJavaScriptProxies = false };
    app.MapSignalR(configuration);

    var connectionString = SignalRHostConfiguration.ServiceBusConnectionString;
    var topicName = SignalRHostConfiguration.ServiceBusNotificationTopic;
    var topicSubscriptionName = SignalRHostConfiguration.TopicSubscriptionName;

    var client = SubscriptionClient.CreateFromConnectionString(connectionString, topicName, topicSubscriptionName);

    client.OnMessage(message =>
    {
        var notificationHubContext = GlobalHost.ConnectionManager.GetHubContext<AbilityNotificationHub>();
        notificationHubContext.Clients.All.showNotificationInPage(message.GetBody<String>());
    });
}

{% endhighlight %}

<h4>The NotificationDispatcher Project</h4>

From this project, we want to send custom messages to service bus topic so that above mentioned SignalR host project can listen to those messages. The stateless service provides us entry point to run our service and we will be using it as below to send custom message every 15 seconds.

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
        var message = new BrokeredMessage("This is a test message generated on " + DateTime.Now.ToUniversalTime().ToString("MM/dd/yyyy HH:mm:ss.fff",
                        CultureInfo.InvariantCulture));
        client.Send(message);

        await Task.Delay(TimeSpan.FromSeconds(15), cancellationToken);
    }
}
{% endhighlight %}

<h4>The NotificationListener - a web Project</h4>

In this project, signalr javascript client is used to connect with signalr host and as soon as message is received, we are appending those messages in web page. Following is the code snippet where we are connecting with the host and reading the received messages.

{% highlight js %}
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

[servicefabricintro]:   https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-overview
[microservices]: /blog/2017/what-is-micro-services-architecture/
[github]: https://github.com/prajapatin/SignalROnServiceFabric
[servicebus]: https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions
[servicefabric]: https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started