---
layout: post
title:  "Operationalizing the Machine Learning model inferencing with an engineering approach"
description: "An engineering approach to operationalize the machine learning model inferencing using open ML model format - ONNX."
date:   2020-08-17 23:30:00
comments: true
image: /images/Operationalize ML Model using ONNX.png
categories: [Architecture, Data Analytics]
keywords: "Architecture, Analytics, ONNX"
---
<h3>What is machine learning model inferencing?</h3>

Once you train a machine learning model to solve specific business problem, you need to think about how you would be consuming that model in production. The machine learning model inferencing is to generate the score and predict the output using machine learning model. While the machine learning (ML) training environment is used to train the ML Model, the ML inference environment is where you operationalize your model to generate the score from your ML model. 

<h3>Challenges faced in machine learning model inferencing</h3>

There are various challenges which require an engineering approach to come up with production grade solutions, following is the brief on those challenges.

**_Plethora of Machine Learning Tools/Libraries/Frameworks_**

Data Scientists use various libraries like SKLearn, Keras, Pytorch etc. to train machine learning models which in turn need different ineferencing environments. If the ML model is trained using SKLearn library then to generate score using that model requires SKLearn runtime environment. The challenge here is about lock-in with one framework/tool ecosystem and it becomes difficult to create common inference environment. 

**_Requirement of different ML inferencing environments_**

The use cases of AI/ML have been wide-spread in various industries & domains having different requirement for ML model deployment and operations. The IIoT industry has different demand to deploy machine learning model because ML model inferencing has to happen in low configuration device or some time in embedded device. Some use cases demand hosting model in cloud environment with high scalability requirement while many uses cases demand running ineferencing environment on edge server. The challenge here is that yo need to prepare your runtime environment for such different targets with keeping in mind scalability and performance. The challenge is also about  dependency on specific tools/libraries which were used to train the model because the runtime environment (i.e. embedded device) may not be able to support those tools and libraries.

**_Dynamic Scalability requirement in ML Inferencing_**

The ML model inferencing demnads very high performance and scalability in certain situtaion so the runtime environment must be capable of utilizing hardware acceleration or should be able to run in low memory footprint. So preparing inference runtime with dynamic scalability and performance is another challenge, particularly to support various inference runtime targets mentioned in above paragraph.


<h3>Operationalizing ML Models using ONNX</h3>

To overcome challenges mentioned, there needs to be a common ML inference environment without needing library/tool specific dependencies. To have such common inference runtime, there needs to be a common format for machine learning model so that models trained using various libraries can be converted to common format and can be used for scoring using common inference environment. There is such open format for machine learning model, which is called [ONNX](https://onnx.ai/).

**What is ONNX?**

**What is ONNX Runtime?**

<h4>Proposed Architecture</h4>

<image src="/images/Operationalize ML Model using ONNX.png"></image>

