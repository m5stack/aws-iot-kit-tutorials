+++
title = "智慧空间"
chapter = true
weight = 50
pre = "<b>4. </b>"
+++

在此教程中，您将把上一个模块 **智能恒温器** 中的智能恒温器解决方案扩展为 **智慧空间** 解决方案。智慧空间是指利用有关空间的见解来改进空间或为空间增加功能。您将利用分析和机器学习功能，根据有关部署智能恒温器的空间的占用情况原始数据得出预测结果。智慧空间解决方案将指导您根据恒温器数据创建新的机器学习模型，以及改进空间分类方法，以便更好地操控智能恒温器。

在开始实验之前，请验证以下先决条件：
1. 您有一个 M5Stack Core2 ESP32 IoT for AWS IoT EduKit 开发套件。
2. 您有一个未运行任何生产工作负载的 AWS 账户（即账户可用于沙箱和评估目的）。
3. 您有一个用户或角色，可以登录具有管理员访问权限的 AWS 账户。
3. 您的 Core2 for AWS IoT EduKit 已成功在 AWS IoT Core 中预置，并且已通过 MQTT 与 AWS 进行通信。如果您还没有完成基本功能的设置，请先学习 [**智能恒温器**](/cn/smart-thermostat.html) 教程。

学习目标。完成此实验后，您应了解：
1. 如何将设备遥测数据转发到 AWS IoT Analytics 以进行存储、转换并创建分析数据集。
2. 如何利用 Amazon SageMaker Studio 创建实验，该实验可以根据数据集自动生成机器学习模型。
3. 如何将机器学习模型部署到可供使用的 API 终端节点上。
4. 无服务器函数是如何使用机器学习推理 API 来增强应用程序的。

让我们从 [**简介**](/cn/smart-spaces/introduction.html) 部分开始学习。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}