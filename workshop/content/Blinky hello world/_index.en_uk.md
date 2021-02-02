+++
title = "Blinky Hello World"
chapter = true
weight = 20
pre = "<b>2. </b>"
+++

Learn how to create a "blinky" (the hello world for microcontrollers) application with your M5Stack Core2 for AWS IoT EduKit reference hardware. Using your own AWS account, you'll walk-through how to connect to AWS IoT Core, send and receive messages, and view the messages on both the device and in AWS cloud. All of the steps and skills used here will provide the foundation to be successful in subsequent tutorials. Specifically, you will:
- Install and configure additional necessary software.
- Register a "thing" in AWS IoT Core using the security certificates pre-provisioned on the onboard secure element.
- Connect and send MQTT messages from the reference hardware to AWS IoT Core using the ESP-IDF (FreeRTOS kernel with symmetric multiprocessing).
- Receive an MQTT message on the reference to trigger blinking an LED. 
 
All the content in this tutorial assumes you presently have the [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT EduKit](https://www.amazon.com/dp/B08NP5LVFH) in your possession, an [AWS account](https://signin.aws.amazon.com/signin) that isn't running production workloads, and are comfortable with basic technical concepts and tools—such as the command prompt/terminal. To purchase your own kit, check out [Amazon.com](https://www.amazon.com/dp/B08NP5LVFH), the [M5Stack store](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) or their global distributors. If you haven't registered for an AWS account yet, please [create an account](https://portal.aws.amazon.com/billing/signup).


To get started with this tutorial, go to the first chapter, [**Prerequisites**](/en_uk/blinky-hello-world/prerequisites.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}