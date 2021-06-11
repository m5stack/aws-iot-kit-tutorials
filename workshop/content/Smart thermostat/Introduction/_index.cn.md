+++
title = "简介"
weight = 10
pre = "<b>a. </b>"
+++

## 您的任务
在此场景中，您将担任全栈开发人员，负责自动化会议室的恒温器功能，从而节约能源。您将使用 Core2 for AWS IoT EduKit 参考硬件模拟恒温器硬件，并将参考硬件工具包作为 HVAC 控制器，与 AWS 云的强大功能结合，部署端到端解决方案。我们假定一个虚构的 HVAC 系统，并将 Core2 for AWS IoT EduKit 作为恒温器。

当空间内有员工时，恒温器应该使用较窄的温度范围，以保证员工尽可能地舒适。当空间内没人时，恒温器可以使用较宽的温度范围，从而节约能源。该解决方案要能识别空间内何时有员工，从而启动 HVAC 提供舒适的温度。

## 解决问题
为了检测空间内何时有人，代替所需的摄像设备，该解决方案将使用麦克风工具包来采集简单的声音强度，这提供了最小的成本。您将通过麦克风获取周围的声音强度，通过设备传感器获取空间内温度，然后将这些值发布给 AWS。

您在 AWS 中的无服务器解决方案会将声音强度转化为布尔值 (true or false)，用它来确认空间内是否有人。您可以使用简单的声音强度阈值来进行确认。

如果空间内有人且测量温度超出了舒适温度范围，解决方案将向设备发送命令，根据需要开始制热或制冷。如果后续测量的温度回到了舒适范围，解决方案将向设备发送命令，恢复待机模式。

## 解决方案架构
![智能恒温器解决方案架构](introduction/thermostat-overview.png)

## 继续学习！
准备好开始构建了吗？ 下面来看看您是否具备以下几种先决条件：

1. 您是否已经完成了 **Blinky Hello World（连接到云的 Blinky）** 教程？ 
2. 您是否已经在 AWS IoT Core 中预置了 Core2 for AWS IoT EduKit？ 也就是说，您是否使用证书和附加的策略将设备注册为 AWS IoT 事物并且可支持您执行发布和订阅操作？ 这些步骤位于 **Blinky Hello World（连接到云的 Blilnky）** 教程中。
3. 您是否确认过自己可以通过使用测试 MQTT 客户端（就像 AWS IoT Core 控制台中的 test 客户端一样）看到来自的设备的消息？ 您应该能够订阅设备发布的主题并看到这些消息到达测试客户端。
4. 您是否知道您的 Core2 for AWS IoT EduKit 挂载的串口？ 这也在 **Blinky Hello World（连接到云的 Blilnky）** 教程中进行过说明。

如果您符合上述条件，就可以继续学习下一章节，[**数据获取**](/cn/smart-thermostat/data-acquisition.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}