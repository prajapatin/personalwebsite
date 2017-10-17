---
layout: post
title:  "Serverless architecture with Microsoft Azure"
description: "Serverless architecture options currently available and how Microsoft Azure is a good alternative"
date:   2017-10-16 23:15:00
comments: true
image: /images/serverless.png
categories: [Cloud, Architecture]
keywords: "serverless, cloud, azure, architecture"
---
<h3>What is Serverless Architecture?</h3>

If we google for "serverless architecture" phrase, we will find plenty of information with multiple definitions. So instead of providing definition, I would like to walk you through little bit of history of computing and how serverless architecture came into existence. 

1. Phase-1
Before cloud computing came in, we used to develop and host software applications ourselves. We all know that it was not only about developing a software but making sure that hosting is taken care of by us; which involved securing hardware resources, managing scalability for higher demand, patching server operating systems etc. It was very difficult to satisfy dynamic load/compute requirement and cost was sky rocketing as we needed to buy hardware resources upfront considering future requirement.

2. Phase-2
This was the phase where cloud computing came into picture but more as an Infrastructure-as-a-Service (IaaS) offering so some of the pains mentioned in phase-1 gone away. You no longer maintained your hardware in this phase and what you worried about is your software. This was the phase where we used to take hardware on lease/rent based on need basis and de-provisioned them if demand goes down. We needed to still manage hardware resources but without actually owning it permanently. 

3. Phase-3
This is where SaaS (Software as a Service) and PaaS (Platform as a Service) introduced very flexible software/application development life cycle. You no longer worry about hardware and underlying operating system, also you get a platform on which you can just develop your business applications. You get everything apart from your business logic/app so time we used to spend on hardware/OS, can be used to develop the application (in SaaS, you even do not develop the application but start using it from browser). We still spend time here to allocate resources based on demand (obviously as an automated provisioning method) and can control the overall cost but not as a lean management. What I mean here is that you automatically allocate resources and pay for it until explicitly you de-provision it and some of the PaaS services will force you to select certain plan when you provision it.

4. Serverless Computing
I do not call it a phase because it is still part of a cloud computing, the difference here is that you run your certain logic when you need to and vendor will charge you only for the time your logic has run. If we take an example of one web service call: when the HTTP request reaches to your serverless code/logic; your serverless compute starts, serves the request and shuts down itself once response is sent back. So, now you can see that how you are being charged is for the time your code is executed. There are still servers involved here but it is no longer your concern as serverless computing makes sure that your scalability demand is always satisfied regardless of number of servers involved to serve/execute your logic. The serverless computing vendors actually will use small containers (e.g. docker containers) to run your logic when there is a request/trigger for it (You can read about containers in one of my [previous articles][windowsservicecontainer]).

The serverless computing is not going to be completely new paradigm and it will co-exist with other traditional architectures. There are specific use-cases where you can use serverless computing, the examples of serverless computing are [AWS Lambda][awslambda], [Azure Functions][azurefunctions], [Webtask][webtask]. I will focus on Azure Functions in this article.

<h3>When shall we use serverless computing?</h3>

Following are the scenarios when serverless computing can be used
  
  1. When you want to run certain logic in specific time interval
  2. Certain services require very high scalability compared to rest of your services can be      hosted as a serverless computing
  3. When you want to run your logic on certain event
  4. When you use third-party services/apis as an integration, you can use serverless             computing for certain types of triggers

  We need to keep in mind that there are limitations of serverless computing as you cannot install any monitoring applications and you get very limited monitoring/logging support (it is bettering day by day though), you do not have a choice on memory or CPU (as it is a serverless ;)), vendor will impose some kind of limitations like maximum number of serverless computes can be executed in parallel for your account etc.

<h3>Azure Functions</h3>

Azure Functions is Serverless computing offering from Microsoft. 

<image src="/images/serverless.png"></image>


[windowsservicecontainer]: /blog/2017/windows-service-as-a-container-on-service-fabric/
[webtask]: https://webtask.io/
[awslambda]: https://aws.amazon.com/lambda/
[azurefunctions]: https://azure.microsoft.com/en-us/services/functions/