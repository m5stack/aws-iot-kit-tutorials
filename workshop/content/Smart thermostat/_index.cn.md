+++
title = "智能恒温器"
chapter = true
weight = 40
pre = "<b>3. </b>"
+++

在此教程中，您将参考硬件配置成为一个智能恒温器，来控制一个虚构的暖通空调 (HVAC) 系统。智能恒温器会将测量到的室温和声音强度报告给它在云中的设备影子。您还需要配置一个无服务器应用程序，用于侦听上报的测量值、确定恒温器应设置的状态，以及向设备发送命令，告诉它应该做什么。

假设。在开始学习此教程之前，请验证以下先决条件：
1. 您有一个 [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT Kit](https://www.amazon.com/dp/B08VGRZYJR/)。
2. 您有一个未运行任何生产工作负载的 AWS 账户（即账户可用于沙箱和评估目的）。
3. 您有一个用户登录名或角色，可以登录具有管理员访问权限的 AWS 账户。
4. 您的 Core2 for AWS IoT Kit 已成功在 AWS IoT Core 中预置，并且已通过 MQTT 与 AWS 进行通信。如果您没有完成预置，也没有建立连接，请先学习 [**Blinky Hello World**](/cn/blinky-hello-world.html) 教程。
5. 您至少要在技术方面对 AWS IoT 消息概念（例如主题、发布和订阅）有一个基本的了解。

学习目标。完成此实验后，您应了解：
1. 如何从 Core2 for AWS IoT Kit 设备获取温度和声音强度。
2. 如何将温度和声音测量结果从设备发送到 AWS IoT Core。
3. 如何将测量值报告给设备影子。
4. 如何使用 AWS IoT Core 规则引擎执行消息转换。
5. 如何构建可响应输入信息并检测复杂事件的无服务器应用程序。
6. 如何通过设备影子将命令发送给设备。

要开始学习此教程，请进入第一章，[**简介**](/cn/smart-thermostat/introduction.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}