+++
title = "数据路由和存储"
weight = 20
pre = "<b>b. </b>"
+++

## 章节简介
学完本章节后，您的无服务器应用程序应该能够：

* 将从智能恒温器接收到的消息转发到托管的存储和分析服务。
* 针对处理过的数据进行查询，生成具体化的结果视图。

## IoT 数据存储和分析的概念
到目前为止，您的 IoT 解决方案的每一个方面都是临时性的，因为每一条消息都是经过接收、处理，然后丢弃的。在 IoT Events 检测器模型这个例子中，有一个有状态的实体来响应新消息，但除此之外，没有存储恒温器消息的历史记录。目前进行的是构建恒温器消息数据存储的步骤。

AWS 提供了许多在云中存储数据的方法，也提供了很多 IoT 数据存储方法。该解决方案推荐使用 [AWS IoT Analytics](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html) 服务，这是一项托管服务，专门用于批量接收、存储、处理和分析 IoT 数据。您将使用 IoT Analytics 来存储每个设备影子上的更新。

AWS IoT Analytics 文档对此进行了更详细的介绍，但这里简要介绍一下它的工作原理。该服务的接入点是一项名为 “channel（通道）”的资源。通道可以存储工作流程的所有原始数据。它还会将接收到的每个消息的副本发送到下一个资源，即 “pipeline（管道）”。管道是指在分析使用案例中使用数据之前处理、清理、筛选和丰富数据的一系列活动。经过处理的消息将从管道转移到 data store（数据存储）。与通道类似，数据存储是一种长期存储单元，适用于经过处理的数据。最后，data set（数据集）是 AWS IoT Analytics 项目中的最后一项资源。数据集定义了类 SQL 查询，该查询可以将数据存储中的消息以具体化视图的形式读取出来，并将查询内容传递到 S3 存储桶等目的地。

您的智慧空间解决方案将在 AWS IoT Analytics 中积累来自恒温器的多个小时的运行数据。之后，数据集查询的结果将使这些数据可用于由 Amazon SageMaker 提供的机器学习工具链。

[AWS IoT Analytics](https://aws.amazon.com/iot-analytics/) 的功能特性不胜枚举，但就本模块而言，它用于存储恒温器消息的历史记录，并是将消息汇总为机器学习模型的训练数据集的最简单方法。

{{% notice warning %}}
此应用程序运行操作 6 个小时可能会产生 AWS 上的费用。推荐在您完成教程之后执行 [清理步骤](/cn/smart-spaces/conclusion.html#clean-up) 避免成本的支出。
{{% /notice %}}

## 如何设置无服务器基础设施
下面的步骤详细介绍了如何创建新的 IoT Core 规则和新的 AWS IoT Analytics 项目，以及如何使用规则将设备影子消息转发到 AWS IoT Analytics 项目。IoT Core 规则创建向导中有一个便捷界面，可代表您创建整个 AWS IoT Analytics 项目！

1. 在 [AWS IoT Core 控制台](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/) 中，依次选择 **Act（操作）**、**Rules（规则）** 和 **Create（创建）**。
2. 为规则命名（例如 `storeInIoTAnalytics`）并提供说明。
3. 使用以下查询：请务必用 Core2 for AWS IoT Edukit 参考硬件工具包屏幕上打印的客户端 Id/序列号替换 **<<CLIENT_ID>>**。

```SQL
SELECT current.state.reported.sound, current.state.reported.temperature, current.state.reported.hvacStatus, current.state.reported.roomOccupancy, timestamp FROM '$aws/things/<<CLIENT_ID>>/shadow/update/documents'
```

4. 对于 *Set one or more actions（设置一个或更多操作）*，选择 **Add action（添加操作）**。
5. 选择 *Send a message to IoT Analytics（将消息发送到 IoT Analytics）*，然后选择 **Configure action（配置操作）**。
6. 选择 *Quick create IoT Analytics resources（快速创建 IoT Analytics 资源）*，并为 *Resource prefix（资源前缀）* 提供一个项目名称。本模块的后续步骤中假定前缀为 `smartspace`。选择 **Quick Create（快速创建）**，所有 AWS IoT Analytics 资源将会自动创建并配置。
7. 选择 **Add action（添加操作）** 以完成此操作配置并返回规则创建表。
8. 选择 **Create rule（创建规则）** 以完成新规则的创建。

## 验证步骤
在进入下一章节之前，您可以验证您的无服务器应用程序是否已按预期配置完毕，方法是：

1. 请确保智能恒温器已接通电源、正在发布数据且已部署在您要使用的房间。
2. 通过 AWS IoT Analytics 控制台，查看最新的数据集内容，并验证是否有环境声音强度、温度和空间占用情况的历史记录。要检查此类内容，请在 [IoT Analytics 控制台](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets) 中找到您的数据集，选择 **Actions（操作）** 和 **Run now（立即运行）**，然后等待 *Result preview（结果预览）* 更新为最新内容。您应该可以看到类似于以下内容的结果：

{{< img "iot_analytics-dataset_run.en.png" "Running the data set" "1 - Actions（操作）, 2 - Run now（立即运行）" >}}
{{< img "iot_analytics-dataset_preview.en.png" "Preview of the data set" >}}

如果没有问题，我们就进入 [**机器学习**](/cn/smart-spaces/machine-learning.html) 部分。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}