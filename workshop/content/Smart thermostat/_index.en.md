+++
title = "Smart Thermostat"
chapter = true
weight = 40
pre = "<b>3. </b>"
+++

In this tutorial, you will configure your reference hardware into a smart thermostat that controls a fictional HVAC system. Your smart thermostat will report the measured room temperature and noise level to its device shadow in the cloud. You will also configure a serverless application that will listen for the reported measurements, determine the state to which the thermostat should be set, and send commands back to the device that tells it what to do do. 

Assumptions. Before starting this tutorial, verify the following prerequisites:
1. You have an [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT EduKit](https://www.amazon.com/dp/B08VGRZYJR/).
2. You have an AWS account that is not running any production workloads (i.e. an account safe for sandbox and evaluation purposes).
3. You have a user login or role to the AWS account with administrator access.
4. Your Core2 for AWS IoT EduKit has been provisioned in AWS IoT Core and is already communicating with AWS via MQTT. Start with the [**Blinky Hello World**](/en/blinky-hello-world.html) tutorial if you have not completed provisioning and established connectivity.
5. You have at least a basic technical understanding of AWS IoT messaging concepts such as topics, publishing, and subscribing.

Learning Objectives. By the end of this lab, you should know:
1. How to acquire temperature and sound levels from the Core2 for AWS IoT EduKit device.
2. How to publish temperature and sound measurements from the device to AWS IoT Core.
3. How to report measured values to the device shadow.
4. How to perform message transforms with the AWS IoT Core rules engine.
5. How to build a serverless application that responds to inputs and detects complex events.
6. How to send commands to your device via the device shadow.

To get started with this tutorial, go to the first chapter, [**Introduction**](/en/smart-thermostat/introduction.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}