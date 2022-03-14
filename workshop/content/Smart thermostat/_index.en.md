+++
title = "Smart Thermostat"
chapter = true
weight = 40
pre = "<b>3. </b>"
+++

In this tutorial, you configure your AWS IoT EduKit to simulate a smart thermostat that controls a fictional HVAC system. Your smart thermostat reports the measured room temperature and noise level to its [device shadow](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html) in the cloud. You also configure a serverless application that listens for the reported temperatures, determines the state to set the thermostat, and sends commands back to the device. 


Assumptions. Before starting this tutorial, verify the following prerequisites:
1. You have an [M5Stack Core2 ESP32 IoT Development Kit for {{<awsEdukitShort-en>}}](https://www.amazon.com/dp/B08VGRZYJR/).
2. You have an AWS account that is not running any production workloads (i.e. an account safe for sandbox and evaluation purposes).
3. You have a user login or role to the AWS account with administrator access.
4. Your Core2 for {{<awsEdukitLong-en>}} has been provisioned in AWS IoT Core and is already communicating with AWS via MQTT. Start with the [**Blinky Hello World**](/en/blinky-hello-world.html) tutorial if you have not completed provisioning and established connectivity.
5. You have at least a basic technical understanding of AWS IoT messaging concepts such as topics, publishing, and subscribing.

Learning Objectives. By the end of this lab, you should know:
1. How to acquire temperature and sound levels from the Core2 for {{<awsEdukitShort-en>}} device.
2. How to publish temperature and sound measurements from the device to AWS IoT Core.
3. How to report measured values to the device shadow.
4. How to perform message transforms with the AWS IoT Core rules engine.
5. How to build a serverless application that responds to inputs and detects complex events.
6. How to send commands to your device via the device shadow.

This tutorial requires that you have:
- The [M5Stack Core2 for AWS IoT EduKit reference hardware](https://www.amazon.com/dp/B08VGRZYJR/) (AWS IoT EduKit).
- An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) that is *not* running production workloads.
- A user in your AWS account that is assigned to you and has administrator access.
- Completed the [Cloud Connected Blinky](/en/blinky-hello-world.html) tutorial.
- Confirmed which serial port the AWS IoT EduKit device uses.
- Are comfortable with basic technical concepts and tools; such as, the command prompt or terminal window.

### Learning Objectives

By the end of this tutorial, you will:
1. Acquire temperature and sound levels from the AWS IoT EduKit.
1. Publish temperature and sound measurements from the AWS IoT EduKit to AWS IoT Core.
1. Report measured values to the AWS IoT device shadow.
1. Perform message transformations with the AWS IoT rules engine.
1. Build a serverless application that responds to inputs and detects complex events.
1. Send commands to your device through the device shadow.

To begin, continue to [**Introduction**](/en/smart-thermostat/introduction.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}