+++
title = "连接到云的 Blinky"
chapter = true
weight = 30
pre = "<b>2. </b>"
+++

在电影中，Blinky 是双眼光芒闪烁的机器人，在本章节中，您将了解如何使用 M5Stack Core2 for AWS IoT EduKit 参考硬件创建 “Blinky” 应用程序（微控制器的 [Hello world](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program)）。您将使用自己的 AWS 账户来演练如何将您的设备连接到 [AWS IoT Core](https://aws.amazon.com/iot-core/)、如何发送和接收 [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) 消息，以及如何在设备和 AWS 云端查看消息。这里采用的所有步骤和技能都将为成功学习后续教程打下良好的基础。具体而言，您将：
- 在您的主机上安装并配置 [AWS CLI](https://aws.amazon.com/cli/)，AWS CLI 可以用来远程管理您的 AWS 服务。
- 使用提前预置的安全证书在 AWS IoT Core 中注册 “[事物](https://docs.aws.amazon.com/iot/latest/developerguide/iot-thing-management.html)”。
- 配置您的参考硬件，将其连接到 Wi-Fi，并连接到您到 AWS 账号的 [AWS IoT 终端节点(endpoint)](https://docs.aws.amazon.com/general/latest/gr/iot-core.html)，发送 MQTT 消息到 AWS Iot Core上。
- 在参考硬件上接收来自云端的 MQTT 消息，该消息会触发 LED 闪烁。
 
本教程的所有内容假定了您目前已经拥有 [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT EduKit](https://www.amazon.com/dp/B08VGRZYJR/) 和并 *不* 运行生产负载的 [AWS 账户(account)](https://console.aws.amazon.com/console/home)，还按照入门教程 [配置](getting-started/prerequisites.html) 完成了您的环境，并且了解基本的技术概念和工具（例如命令提示符/终端）。要购买工具包，请通过 [Amazon.com](https://www.amazon.com/dp/B08VGRZYJR/)、[M5Stack 店铺](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) 或其相关全球经销商进行购买。如果您还未注册 AWS 账户，请 [创建账户](https://portal.aws.amazon.com/billing/signup)。


要开始学习此教程，请进入第一部分 [先决条件](blinky-hello-world/prerequisites.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}
