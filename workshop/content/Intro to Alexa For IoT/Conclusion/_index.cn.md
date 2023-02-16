
+++
title = "总结"
weight = 50
pre = "<b>e. </b>"
+++

## 总结
您已经学完了有关运行 Alexa for AWS IoT (AFI) 的 AWS IoT Kit 动手实践教程，了解了如何使用自定义移植的 Espressif Voice Assistant SDK 和 Alexa，通过 Alexa Smart Home 命令控制板载外围设备。有关适用于 AVS 的智能家居的更多信息，请参阅 [相关文档](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/smart-home-for-avs.html) 或 [博客文章](https://developer.amazon.com/en-US/blogs/alexa/device-makers/2020/04/create-a-sample-alexa-built-in-disco-ball-with-smart-home-for-av)。

如果您想进一步了解其他可用接口，或者想要自行利用 M5Stack Core2 for AWS IoT 参考硬件的其他功能来实现一些用途，请查看 [ModeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-modecontroller.html) 和 [ToggleController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-togglecontroller.html)。运用您的创造力和在本教程其他章节中学到的知识来探索语音的新功能！

这是最后一个教程。我们会发布更多内容，但现在，您已经能够使用到目前为止所学的技能来通过 AWS 服务构建自己的 IoT 解决方案了。

## 清理
在这个解决方案中，您没有使用自己的任何 AWS 资源。您可以通过在 [PIO CLI 终端窗口](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window) 中输入以下命令来擦除设备软件（并从闪存中删除证书）：
```bash
pio run --environment core2foraws --target erase
```

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}