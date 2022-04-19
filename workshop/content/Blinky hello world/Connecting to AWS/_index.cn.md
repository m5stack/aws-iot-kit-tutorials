+++
title = "连接到 AWS IoT Core"
weight = 30
pre = "<b>c. </b>"
+++

在本章节中，您将配置、构建和烧录设备固件，以将设备连接 Wi-Fi 网络并连接到 AWS IoT Core。为了连接到 AWS IoT Core 并与之通信，您需要使用 Wi-Fi 凭证和 [AWS IoT终端节点](https://docs.aws.amazon.com/iot/latest/developerguide/connect-to-iot.html#iot-device-endpoint-intro) 的 URL 来配置设备。通过使用 [AWS IoT Device SDK for Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) 和 [Microchip ATECC608 Trust&GO](https://www.microchip.com/wwwproducts/en/ATECC608B-TNGTLS) 中预置的证书可以简化安全 MQTT 连接的建立。通过使用安全元件，您无需从 AWS IoT Core 检索证书或生成自己的证书，AWS IoT Device SDK for Embedded C 包含的连接库简化了连接以及对 AWS 服务和功能的访问。

## 配置 ESP32 固件
您将通过 [Kconfig](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html) 配置源代码。Kconfig 是 Linux 内核使用的配置系统，有助于将可用的配置选项（symbols）简化为树形结构。

您可以从仓库的 **Blink-Hello-World** 目录进入配置菜单：
```bash
pio run --environment core2foraws --target menuconfig
```

{{< img "idf_menuconfig-wifi.en.webp" "Configuring Core2 for AWS IoT EduKit with p.py menuconfig" >}}

下面，您将设定配置。使用键盘上的方向键转到  然后，从菜单中选择 **AWS IoT EduKit Configuration（AWS IoT EduKit 配置）**。使用 Wi-Fi 凭证设置 **Wi-Fi SSID** 和 **Wi-Fi Password**。完成后，按键盘上的 *S* 键进行保存，按 *enter* 键确认文件的位置，然后按 *q* 键退出。

{{% notice warning %}}
确保您的 SSID 用于 2.4GHz 网络。M5Stack Core2 for AWS 硬件上的 ESP32-D0WD 不支持 5GHz Wi-Fi 频段。
{{% /notice %}}

## 构建、上传、和监控 Blinky Hello World 固件
现在已经就绪，可以开始构建（编译）和上传烧录 Blinky Hello World 固件。构建、烧写、监控串行输入的操作流程和入门教程中所述相同：
   
1) 要构建固件，使用下面的命令（操作可能需要几分钟的时间）：
    ```bash
    pio run --environment core2foraws
    ```
2) 成功完成构建后，就可以开始通过 USB 接口将编译完成的固件上传到连接到设备中，同时执行命令来监视通过串口输出到主机上的信息:
    ```bash
    pio run --environment core2foraws --target upload --target monitor
    ```
{{% notice info %}}
如果在上传或监控串行输出过程中收到关于不正确端口或超时的错误，请打开 platformio.ini 文件，并按照该文件中的说明手动设置上传端口。
{{% /notice %}}

## 章节总结
在本章节中，您已经成功编译并烧录了设备，还监控了设备的串行输出。使用 AWS IoT Device SDK for Embedded C，参考硬件已经和 [MQTT 消息代理](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) (AWS IoT Core) 完成了认证，做好了接收消息的准备。

现在，您可以继续学习本教程的下一章，[**闪烁 LED**](/cn/blinky-hello-world/blinking-the-leds.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}
