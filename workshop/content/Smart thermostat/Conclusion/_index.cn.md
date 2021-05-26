+++
title = "总结"
weight = 60
pre = "<b>f. </b>"
+++

## 总结
您已经完成了此 AWS IoT EduKit 动手实践教程，将设备作为智能恒温器连接到 AWS，并且部署了简单的应用程序来检测空间占用情况来更新 HVAC 状态变化。

接下来要学习下一个动手实践教程 **智慧空间**：

如果您想通过其他方法来验证您的新的解决方案，那么您可以考虑一种能够增强客户体验的方式。例如，如何修改解决方案才能确保 HVAC 在最短时间内保持运行，而不是在 10 秒内没有检测到环境声音便关闭？ 您是否可以在不为设备更新代码的情况下实现这一点？（我们确信您当然可以。）

## 清理
使用这一解决方案，您在 AWS 中创建了以下资源：

* IoT Core 规则。
* IAM 角色。
* IoT Events 输入和检测器模型。

以上这些资源在不使用时均不会产生任何费用。只有使用资源处理来自设备的新消息时，才会产生费用。如果您要学习下一个动手实践教程 **智慧空间**，我们建议您保留本教程中的所有资源。

利用本教程中的解决方案完成实验后，如果您不打算学习 **智慧空间** 教程，则可以销毁这些资源：

1. 打开 AWS IoT Core 控制台，选择 Act（操作），然后选择 Rules（规则），在列表中找到相应的规则并将其删除。
2. 打开 AWS IoT Events 控制台，选择 Detector models（检测器模型），在列表中找到相应的模型并将其删除。选择 Inputs（输入），在列表中找到相应的输入并将其删除。
3. 打开 IAM 控制台，选择 Roles（角色），在列表中找到与 IoT Core 规则和 IOT Events 检测器模型对应的角色并将其删除。
4. 使用 [PlatformIO CLI 终端窗口](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window) 从 **Smart-Thermostat（智能恒温器）** 文件夹中，擦除设备固件，从而停止设备和 AWS IoT Core 的连接以及消息发送。您也可以按住电源键 6 秒钟来关闭设备：
```bash
pio run --environment core2foraws --target erase
```

下一个要学习的教程是 [**智慧空间**](/cn/smart-spaces.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}