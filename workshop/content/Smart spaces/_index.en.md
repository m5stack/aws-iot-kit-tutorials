+++
title = "Smart Spaces"
chapter = true
weight = 50
pre = "<b>4. </b>"
+++

In this tutorial, you will extend the smart thermostat solution from the previous module, **Smart Thermostat**, into a Smart Space solution. A smart space is the concept of using insights about a space to enhance the space or bring further capabilities to it. You will use analytics and machine learning capabilities to derive predictions from raw data about room occupancy where your smart thermostat is deployed. The Smart Space solution will guide you how to create a new machine learning model from your thermostat data and how to improve the classification of room occupancy to operate your smart thermostat even better.

This tutorial requires that you have:
- The [M5Stack Core2 for AWS IoT EduKit reference hardware](https://www.amazon.com/dp/B08VGRZYJR/) (AWS IoT EduKit).
- An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) that is *not* running production workloads.
- A user in your AWS account that is assigned to you and has administrator access.
- Completed the [Cloud Connected Blinky](/en/blinky-hello-world.html) tutorial.
- Confirmed which serial port the AWS IoT EduKit device uses.
- Are comfortable with basic technical concepts and tools; such as, the command prompt or terminal window.

### Learning Objectives

By the end of this tutorial, you will:
1. Forward device telemetry to AWS IoT Analytics for storage, transformation, and create an analytical data set.
1. Use Amazon SageMaker Studio to create an experiment that automatically generates a machine learning model from your data set.
1. Deploy a machine learning model to a consumable API endpoint.

To begin, continue to [**Introduction**](/en/smart-spaces/introduction.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}