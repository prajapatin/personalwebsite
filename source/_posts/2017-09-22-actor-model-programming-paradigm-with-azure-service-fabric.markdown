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

When we talk about scalability of any software systems/applications, we look at it with two aspects: (1) Whether the system is built with distributed architecture pattern (2) Are we using full capability of the hardware, where the system is hosted?

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

There are many options to implement Actor Model in distributed software architecture, the Erlang is the language built using the concept of actor model. There is an [Akka][akkalink] framework in Java and .Net port of the same framework as an [Akka.net][akkadotnet], which can be used to develop enterprise scale microservices applications. There is a cloud based microservices platform called Azure Service Fabric, about which I have written in [one of my][servicefabric] previous articles. I am going to explain very simple Actor Model based microservices application using Azure Service Fabric.

<h3>Actor Model Service using Azure Service Fabric</h3>

The source code for the sample application can be cloned/downloaded from [here][github]. The application is about setting temperature value through multiple sensor actors and temperature aggregator will collect those values and creates average temperature value. There is no need to spawn cloud cluster for Azure Service Fabric to run this sample application, the local service fabric cluster manager is enough to run and test the application. The information about Azure Service Fabric SDK and local custer manager is available [here][servicefabric].

<h4>Sensor Actor</h4>

The sensor actor has four methods: (1) To Set the temperature (2) To get the temperature (the aggregator actor will call this method to collect the temperature value) (3) To set the index value for each instance of an actor (4) To get the index value for each instance of an actor. We would be using these methods in test project, which shows how to connect with actor and call specific method. The Service Fabric template simplifies the development of Actor by hiding Message sending/receiving functionality (you can look for auto generated ActorEventSource.cs file in each actor project to understand how the message communication logic is implemented) so that we can concentrate on logic. Following code is important in Sensor Actor project.

{% highlight C# %}
Task<double> ISensorActor.GetTemperatureAsync()
{
    return this.StateManager.GetStateAsync<ActorState>("sensorState").ContinueWith(sensorState =>
    {
        ActorEventSource.Current.ActorMessage(this, "Getting current temperature value as {0}", sensorState.Result.Temperature);
        return sensorState.Result.Temperature;
    });
}

Task ISensorActor.SetTemperatureAsync(double temperature)
{
    return this.StateManager.GetStateAsync<ActorState>("sensorState").ContinueWith(sensorState =>
    {
        ActorEventSource.Current.ActorMessage(this, "Setting current temperature of value to {0}", temperature);
        this.StateManager.SetStateAsync<ActorState>("sensorState", new ActorState { Temperature = temperature, Index = sensorState.Result.Index });
    });

}
{% endhighlight %}

In above code we are accessing state manager (it is distributed and provided by service fabric) and retrieving & storing temperature value with specific index. Also we are sending message to current actor through auto generated event/message class.

<h4>Aggregator Actor</h4>

Aggregator actor collects all temperature values from all actors and provides average value, here we can see that how easy is to access specific actor from service fabric through Actor APIs provided.

{% highlight C# %}
public Task<double> GetTemperatureAsync()
{
    Task<double>[] tasks = new Task<double>[1000];
    double[] readings = new double[1000];
    Parallel.For(0, 1000, i =>
    {
        var proxy = ActorProxy.Create<ISensorActor>(new ActorId(i), "fabric:/SensorAggregator");
        tasks[i] = proxy.GetTemperatureAsync();
    });
    Task.WaitAll(tasks);
    Parallel.For(0, 1000, i =>
    {
        readings[i] = tasks[i].Result;
    });
    return Task.FromResult(readings.Average());
}
{% endhighlight %}

In above code we can see that we can access specific actor using id from actor proxy using actor interface and fabric name. It is simple logic to access specific method and rest of the complexities hidden inside service fabric API & runtime.

<h4>Sensor Agrregator Test Project</h4>

This is a console project to test the Actor based microservices application. Once you deploy service fabric project in local cluster, you can run the console application and verify that how you can access actors running inside local service fabric cluster.


[akkalink]: https://akka.io/
[akkadotnet]: http://getakka.net/
[servicefabric]: /blog/2017/signalr-based-app-on-service-fabric/
[github]: https://github.com/prajapatin/ActorPatternOnServiceFabric
