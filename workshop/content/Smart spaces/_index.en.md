+++
title = "Smart Spaces"
chapter = true
weight = 50
pre = "<b>4. </b>"
+++

In this tutorial, you will extend the smart thermostat solution from the previous module, **Smart Thermostat**, into a Smart Space solution. A smart space is the concept of using insights about a space to enhance the space or bring further capabilities to it. You will use analytics and machine learning capabilities to derive predictions from raw data about room occupancy where your smart thermostat is deployed. The Smart Space solution will guide you how to create a new machine learning model from your thermostat data and how to improve the classification of room occupancy to operate your smart thermostat even better.

Assumptions. Before starting this lab, verify the following prerequisites:
1. You have a M5Stack Core2 ESP32 IoT Development Kit for AWS IoT EduKit.
2. You have an AWS account that is not running any production workloads (i.e. an account safe for sandbox and evaluation purposes).
3. You have a user login or role to the AWS account with administrator access.
3. Your Core2 for AWS IoT EduKit has been provisioned in AWS IoT Core and is already communicating with AWS via MQTT. Start with the [**Smart Thermostat**](/en/smart-thermostat.html) tutorial if you have not already completed that baseline of functionality.

Learning Objectives. By the end of this lab, you should know:
1. How to forward device telemetry to AWS IoT Analytics for storage, transformation, and creating analytical data sets.
2. How to use Amazon SageMaker Studio to create an experiment that automatically generates a machine learning model from your data sets.
3. How to deploy a machine learning model to a consumable API endpoint.
4. How a serverless function consumes an ML inference API to augment an application.

Let's begin with the [**Introduction**](/en/smart-spaces/introduction.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}