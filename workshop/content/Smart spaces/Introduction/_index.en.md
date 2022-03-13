+++
title = "Introduction"
weight = 10
pre = "<b>a. </b>"
+++

## Your task

In this scenario, you resume the role of a full-stack developer tasked with automating thermostat functions of a meeting room to conserve energy. Now you are exploring how to improve the mechanism for determining whether the room is occupied over the simple noise threshold used in the previous module. A simple machine learning model could help you identify a more intelligent classification than a static numerical threshold and it can take into consideration the past room occupation data to reduce jitter or false positives.

Training a machine learning model is relatively easy these days with the modern data science tool chain. Training a *good* model, however, is hard and requires expertise, a comprehensive understanding of inputs, and effort over many cycles to optimize it. Today, you will exercise the Amazon SageMaker tool chain to learn how IoT data can be used to train a model, but we can't expect that a *good* model will be produced on the first try.

Please note that completion of this module will span approximately two days' time. While you only have an hour of hands-on work to complete, there are two steps where you will be hands-off. In the first step you deploy your smart thermostat device in the room you want to study and gather telemetry to store for ML training. You will want to deploy your device and gather data for a few hours at least (preferably 24 hours), but the more data you gather, the more accurate the ML model can be. The second, hands-off step will be when the ML model is going through an automated training process after you have gathered sufficient data. This training process can take several hours and we recommend starting it in the morning and returning to it in the afternoon, or letting it run overnight.

## Solution overview

The solution produced in the last module used a static numeric threshold on the incoming sound level to coerce a new key-value called *roomOccupancy*. To train a simple machine learning model to perform a similar function, you will use existing data as a baseline for training. This means you will need to run the existing solution for several hours in a room that has alternating states of the room actively being occupied or not. You will use an aggregated data set of that device telemetry to power an automated ML training experiment that will then be used to classify new device reports with a new *roomOccupancy* value inferred from your trained ML model. Again, your first trained model may not be very accurate, but the purpose of this solution is to give you hands-on experience with storing IoT data and exercising the workflow of training and consuming a new ML model.

## Solution architecture
![Smart Spaces architecture](introduction/smartspace-overview.png)

The workflow that you will deliver in this module has the following key components:

1. A new IoT Core rule will forward device shadow updates to a service called AWS IoT Analytics. The IoT Analytics service stores raw IoT data in bulk, transforms and cleans it to turn raw data into processed data, and provides a query engine to slice processed data for analytical workflows or ML training.
2. Your IoT Analytics project will consist of four resources linked together: a channel for storing raw IoT data from the device shadow, a pipeline to transform/filter/enrich data, a data store for processed data, and a data set that runs saved queries and can send the results for processing.
3. Amazon SageMaker Studio is an integrated machine learning environment where you can build, train, deploy, and analyze your models, all in the same application.
4. An Amazon SageMaker endpoint that hosts your trained model as a consumable API. 
5. An AWS Lambda function that will run some simple code to process your thermostat published messages and make inferences against your new ML model endpoint.

## Let's go!
Are you ready to start building? Let's review you have the following prerequisites sorted:
1. Have you completed the [**Smart Thermostat**](../smart-thermostat.html) module in this set of tutorials? 
2. Have you confirmed that you can see messages arriving from your smart thermostat using a test client like the one in the AWS IoT Core console? You should be able to subscribe to the topic `$aws/things/<<CLIENT_ID>>/shadow/update/accepted` (replacing <<CLIENT_ID>> with your device's client ID/serial number) and see messages arrive in the test client.

If so, let's begin by moving on to the next chapter, [**Data routing and storage**](/en/smart-spaces/data-routing-and-storage.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}