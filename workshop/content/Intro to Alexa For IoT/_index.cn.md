+++
title = "Alexa for IoT 简介"
chapter = true
weight = 60
pre = "<b>5. </b>"
+++

在此教程中，您将在 M5Stack Core2 for AWS IoT EduKit 参考硬件工具包上完成 Alexa Voice Service Integration for AWS IoT (AFI)。您将学习如何使用 Espressif Voice Assistant SDK (VA-SDK) 和 Alexa，通过 Alexa 的智能家居命令控制智能板载 LED。此教程目前使用的是 Espressif 提供的 AWS 账户。这是 Espressif VA-SDK for the M5Stack Core2 for AWS IoT EduKit 参考硬件的 *beta* 端口。

在开始学习此教程之前，请验证以下先决条件：

1. 您有一个 [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT EduKit](https://www.amazon.com/dp/B08VGRZYJR/)。
2. 您已完成 [**入门**](/cn/getting-started.html) 和 [**Blinky Hello World**](/cn/blinky-hello-world.html) 两个教程，您配置完成了您的开发环境，能够编译和上传固件到设备中去。
3. 您至少要在技术方面对 AWS IoT 消息收发概念（例如主题、发布和订阅）有一个基本的了解。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}