+++
title = "总结"
weight = 50
pre = "<b>e. </b>"
+++

## 总结
您已经完成了本动手实践模块，对您的智能恒温器应用程序进行了升级，以利用经过训练的机器学习模型。现在，您已经了解了构建 IoT 应用程序的端到端体验，包括：采集来自传感器的物理数据、同步边缘和云之间的状态、构建无服务器应用程序来控制边缘设备，以及利用机器学习将数据转化为可行的见解。

如果您想要继续扩展此解决方案，可以参考下面的几个建议：

* 向 IoT Analytics 数据集添加一个计时器，让它每天构建一次更新的内容文件。
* 构建一个简单的 Web UI 或预置 Amazon QuickSight，实现 IoT Analytics 中创建的数据集可视化。
* 在 IoT Analytics 中添加一个新的管道活动，通过 Lambda 函数获取当地天气情况，并将其添加到您的恒温器数据中。
* 使用 Core2 for AWS IoT Kit 参考硬件库在 hvacStatus 发生更改时发出声音。
* 使用 Core2 for AWS IoT Kit 参考硬件库在 roomOccupancy 发生更改时闪烁 LED 灯条。

## 清理
在此解决方案和上一个解决方案（**智能恒温器**）中，您成功在 AWS 中创建了以下资源：

* IoT Core 规则。
* IAM 角色。
* IoT Events 检测器模型。
* IoT Analytics 项目（通道、管道、数据存储、数据集）。
* Lambda 函数。
* SageMaker Studio。
* S3 存储桶 （SageMaker Studio 项目的一部分）。
* SageMaker 模型组。
* SageMaker 终端节点。

您的账户将在以下三种情况下继续产生费用。第一，使用预置的计算资源，比如托管在虚拟实例上的 SageMaker 终端节点。这些资源按小时计费。第二，在资源中存储数据，比如 S3 存储桶和 IoT Analytics 项目（通道、数据存储以及经过处理的数据集结果）。这些资源按每月每 GB 等维度计费。第三，执行事件驱动型活动（比如将消息发布到 IoT Core、调用 Lambda 函数以及处理 IoT Events 中的事件），这些活动将按事件计费。因此，如果您要关闭设备，那么事件驱动型活动就会停止。如果您已经使用解决方案完成了一切工作，不需要继续使用这些 AWS 资源了，那么您应该将这些资源销毁。

SageMaker 终端节点是成本最高的资源，因为它是按小时计费的。建议您在使用完此教程后将其销毁。您的模型本身将会一直存在，可以日后重新部署。销毁 SageMaker 终端节点的步骤：

1. 转至 SageMaker 管理控制台，选择 Endpoints（终端节点），在列表中找到相应终端节点并将其删除。

销毁 IoT Analytics 存储资源的步骤：

1. 转至 AWS IoT Analytics，选择 Channels（通道），在列表中找到相应通道并将其删除。
2. 转至 AWS IoT Analytics，选择 Data stores（数据存储），在列表中找到相应数据存储并将其删除。
3. 转至 AWS IoT Analytics，选择 Data sets（数据集），在列表中找到相应数据集并将其删除。

IoT Core 规则、Lambda 函数、IoT Events 输入和检测器，以及 IoT Analytics 管道只会在您使用它们时才会产生更多费用。如果您不再需要这些资源，可以在它们各自的管理控制台中将它们从各自的资源详情页面中删除，从而销毁它们：

1. 转至 AWS IoT Core，依次选择 Act（操作）、Rules（规则），在列表中找到相应规则并将其删除。
2. 转至 AWS IoT Analytics，选择 Pipelines（管道），在列表中找到相应管道并将其删除。
3. 转至 AWS Lambda，选择 Functions（函数），在列表中找到相应函数并将其删除。
4. 转至 AWS IoT Events，选择 Inputs（输入），在列表中找到相应输入并将其删除。
5. 转至 AWS IoT Events，选择 Detector models（检测器模型），在列表中找到相应模型并将其删除。


进入下一个教程： [**Alexa for IoT 简介**](/cn/intro-to-alexa-for-iot.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}