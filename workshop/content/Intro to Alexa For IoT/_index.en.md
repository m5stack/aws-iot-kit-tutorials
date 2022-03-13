+++
title = "Intro to Alexa for IoT"
chapter = true
weight = 60
pre = "<b>5. </b>"
+++

In this tutorial, you will implement Alexa Voice Service Integration for AWS IoT (AFI) on the M5Stack Core2 for AWS IoT EduKit reference hardware kit. You will learn how to use the Espressif Voice Assistant SDK (VA-SDK) and Alexa to control the onboard LED using Alexa Smart Home commands. This tutorial currently uses an AWS account provided by Espressif. This is a *beta* port of the Espressif VA-SDK for the M5Stack Core2 for AWS IoT EduKit reference hardware.

This tutorial requires that you have:
- The [M5Stack Core2 for AWS IoT EduKit reference hardware](https://www.amazon.com/dp/B08VGRZYJR/) (AWS IoT EduKit).
- An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) that is *not* running production workloads.
- A user in your AWS account that is assigned to you and has administrator access.
- Completed the [Cloud Connected Blinky](/en/blinky-hello-world.html) tutorial.
- Confirmed which serial port the AWS IoT EduKit device uses.
- Are comfortable with basic technical concepts and tools; such as, the command prompt or terminal window.
---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}