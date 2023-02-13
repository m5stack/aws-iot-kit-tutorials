+++
title = "闪烁 LED"
weight = 40
pre = "<b>d. </b>"
+++

现在，您的设备软件正在运行，并且您已经连接到 AWS IoT Core，正在发送消息，也已经准备好接收来自云的消息并采取相应行动。在本章节中，您将订阅一个 [MQTT topic（主题）](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html)、查看发布到 AWS IoT Core 的消息，并发送消息让设备 LED 闪烁。由于 MQTT 是 [发布-订阅协议](https://mqtt.org/)，因此您可以订阅和/或发布到特定主题。之前在注册脚本中设置的策略限制了设备可以订阅和发布的主题。这对于筛选至关重要，对于安全性可能也至关重要。您的设备只能发送到或接收与 client Id 相匹配的主题，在这种情况下，client Id 与设备安全单元的唯一序列号相同。

## 通过 AWS IoT MQTT 客户端进行订阅
您可以通过 AWS IoT Core 控制台中的 AWS IoT MQTT 客户端查看和发布 MQTT 消息。首先，请前往 [AWS IoT 控制台](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/)，然后选择 **Test（测试）**，打开客户端页面。

{{< img "aws_iot-mqtt_test_client-subscribe.en.png" "Choose test in AWS IoT console" >}}

在 Subscription topic（订阅主题）区域中，输入 `＃` 订阅所有 MQTT 主题名称。**＃** 是多级通配符 [主题筛选条件](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#topicfilters)，并且只能作为主题筛选条件的最后一个字符使用一次。按 **Subscribe to topic（订阅主题）** 按钮后，您将看到设备发送到云端的消息。设备只能将消息发布到以 `<<CLIENT_ID>>/` 开头的主题。这让其他订阅者（例如，云应用程序）能够针对特定客户端设备筛选主题，并且还可以将范围缩小到更具体的主题（例如，特定设备的温度读数）。

## 闪烁 LED
要让 M5Stack Core2 for AWS IoT Kit 参考硬件侧面的 LED 灯条开始/停止闪烁，我们将从控制台上的 AWS IoT MQTT 客户端发布消息到 Core2 for AWS IoT Kit 订阅的主题。设备订阅的主题会是 `<<CLIENT_ID>>/#` 这样的类型。在设备成功的订阅到该主题之后，你可以在设备上查看到该主题。此外，在主机的串口输出中也可以观察到 client Id。

在 publish（发布）框中，输入下面的内容，其中的 **<<CLIENT_ID>>** 使用您实际的 client Id替换，然后，点击 **Publish**：
```
<<CLIENT_ID>>/blink
```
{{< img "aws_iot-mqtt_test_client-publish.en.png" "Subscribing to messages and publishing with AWS IoT console MQTT client" >}}
您的设备侧面的 LED 灯条现在应该在闪烁。要暂停闪烁，只需再次按 **Publish** 发布消息到同一主题。

{{% notice info %}}
想要退出串口监视，请按下 **CTRL** + **C**。
{{% /notice %}}

## 清理
为了优化使用的资源并避免不必要的潜在 AWS 云服务费用，您将清除设备上的闪存，使其准备好用于后续教程。要清除闪存，您需要进入一个已经构建好的项目（例如，您刚刚完成的项目），正在运行的串行监视终端可能会阻塞端口，按**CTRL** + **C** 退出它，并使用以下命令：

```
pio run --environment core2foraws --target erase
```


进入最后一个章节，[**总结**](conclusion.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}