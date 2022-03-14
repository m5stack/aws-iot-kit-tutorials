+++
title = "Cloud Connected Blinky"
chapter = true
weight = 30
pre = "<b>2. </b>"
+++

Learn how to create a cloud connected "blinky" application (the [hello world](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) for microcontrollers) with your M5Stack Core2 for {{<awsEdukitLong-en>}} reference hardware. Using your own AWS account, you'll walk-through how to connect your device to [AWS IoT Core](https://aws.amazon.com/iot-core/), send and receive [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) messages, and view the messages on both the device and in the AWS cloud. All of the steps and skills used here will provide the foundation to be successful in subsequent tutorials. Specifically, you will:

Welcome to Cloud Connected Blinky. In this tutorial, you create a "blinky" application, which is the equivalent of a [hello world](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) exercise for microcontrollers. The "blinky" exercise demonstrates that you have established communication (telemetry) between the device and the cloud. 

Through this tutorial you connect your device to the [AWS IoT Core](https://aws.amazon.com/iot-core/), send and receive [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) messages, and view the messages on the device and in the AWS cloud. In order to accomplish this, you will:

  - Install and configure the [AWS CLI](https://aws.amazon.com/cli/) on your host machine to remotely manage your AWS services and use provided helper scripts.
- Register a "[thing](https://docs.aws.amazon.com/iot/latest/developerguide/iot-thing-management.html)" in AWS IoT Core using the security certificates pre-provisioned on the onboard secure element.
- Configure your AWS IoT EduKit to connect to your local Wi-Fi, connect to your AWS account's [AWS IoT endpoint](https://docs.aws.amazon.com/general/latest/gr/iot-core.html), and send MQTT messages to the AWS IoT Core.
- Receive an MQTT message from the cloud on the specified reference hardware to trigger blinking an LED. 
 
All the content in this tutorial assumes you presently have the [M5Stack Core2 ESP32 IoT Development Kit for {{<awsEdukitShort-en>}}](https://www.amazon.com/dp/B08VGRZYJR/) in your possession, have an [AWS account](https://console.aws.amazon.com/console/home) that is *not* running production workloads, have your [environment setup](getting-started/prerequisites.html) from the Getting Started tutorial, and are comfortable with basic technical concepts and tools—such as the command prompt/terminal. To purchase your own kit, check out [Amazon.com](https://www.amazon.com/dp/B08VGRZYJR/), the [M5Stack store](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) or their global distributors. If you haven't registered for an AWS account yet, please [create an account](https://portal.aws.amazon.com/billing/signup).

This tutorial requires that you have:
- The [M5Stack Core2 for AWS IoT EduKit reference hardware](https://www.amazon.com/dp/B08VGRZYJR/) (AWS IoT EduKit).
- An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) that is *not* running production workloads.
- A user in your AWS account that is assigned to you and has administrator access.
- Completed the [Getting Started](getting-started/prerequisites.html) tutorial.
- Are comfortable with basic technical concepts and tools; such as, the command prompt or terminal window.


{{% notice note %}}
If you do not have an AWS IoT EduKit, you can purchase one through [Amazon.com](https://www.amazon.com/dp/B08VGRZYJR/), or the [M5Stack store](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) and their global distributors. If you haven't registered for an AWS account, please see [How do I create and activate a new AWS account?](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/).
{{% /notice %}}

To begin, continue to [**Prerequisites**](blinky-hello-world/prerequisites.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}