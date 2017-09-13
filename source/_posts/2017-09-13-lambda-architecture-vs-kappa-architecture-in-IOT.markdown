---
layout: post
title:  "Lambda Architecture Vs Kappa Architecture in IoT"
description: "Lambda Architecture & Kappa Architecture are major architecture patterns used in IoT for analytics."
date:   2017-09-13 21:00:00
comments: true
image: /images/signalr.png
categories: [Architecture, IoT, Data Analytics]
keywords: "Architecture, Lambda Architecture, Kappa Architecture, IoT, Analytics"
---
<h3>Lambda Architecture & Kappa Architecture use case in IoT</h3>

In IoT world, the large amount of data from devices is pushed towards processing engine (in cloud or on-premise); which is called data ingestion. The scenario is not different from other analytics & data domain where you want to process high/low latency data. The data ingestion and processing is called pipeline architecture nad it has two flavours as explained below. 

<h3>Lambda Architecture</h3>

In Lambda Architecture, there are two data paths as mentioned below

  1. The 'hot' path: In this pipeline, high latency data flows for rapid consumption by analytics client. The        event/trigger data from IoT devices is a good use case in IoT domain.
  2. The 'cold' path: In this pipeline, data goes and processed in batches and usually data can tolerate             latency. The data gets processed and stored for a longer duration for different type of analytics like          Predictive analytics.

In 'cold' path, data usually would be immutable so any changes in data must be stored with a new value along with timestamp. Now you can imagine that any type of data along with it's history will have many use cases for IoT domain. You can look for a data in specific time frame and predict the maintainance of machines/devices or any use cases where you need to be as accurate as possible and you have a freedom to take time to process the data. While in 'hot' path, the data would be mutable and can be changed in place when data is moving in pipeline from one process to another. The result of processing should be in real time or near real time so you may have restriction on types of calculation you can do in this pipeline. You can get some kind of parameter (e.g. temperature) anomalies in this processing where you have a little freedom in accuracy and you can run different types of algoroithms which can provide approximation in values. 

The 'hot' and 'cold' paths ultimately converges at the client application and client decides how to consume specific type of data. Clients can choose to use less accurate but most recent data through hot path or can go ahead with less timely and more accurate data through cold path of the Lambda Architecture. The Lambda Architecture is resilient to the system failure as there is always original data available to recompute to come up with desired output.

<h3>Kappa Architecture</h3>

The Kappa Architecture suggests to remove cold path from the Lambda Architecture and allow processing in always near realtime. The Kappa Architecture is a brain child of Linkedin's engineering team, they came up with this solution to avoid code sharing between two different paths (hot and cold). Usually in Lambda architecture, we need to keep hot and cold pipelines in sync as we need to run same computation in cold path later as we run in hot path. The data in pipeline called events and good example of event is the change in temperature so new temperature value from specific device will become new value of the datum without changing the previous datum.

The unified data/logs Queue would be fault tolerant and would be distributed in nature (e.g. Apache Kafka, Azure Service Bus etc.). To support fault tolarance, the data would be persisted to some kind of fault tolarant & distributed permanant storage. The Kappa architecture is similar to CQRS (command query responsibility segregation) pattern so if you are aware of it, you will find quite similarity with it.

<h3>Which architecture pattern to choose when?</h3>

There are many arguments against each other while choosing one of the patterns and it is very tough to come to conclusion on which one is better. The decision to choose one among two should be completely dependent on use case, needs and choice. 

