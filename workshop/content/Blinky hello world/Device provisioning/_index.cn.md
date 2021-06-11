+++
title = "设备预置"
weight = 20
pre = "<b>b. </b>"
+++

在本章节中，您将预置您的设备，使用板载 Microchip ATTECC608 Trust&GO 安全元件通过建立 [TLS 连接](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html) 连接到 AWS IoT Core。内置的硬件信任根让您可以在不公开私有密钥的情况下简化和加快预置流程。您可以检索设备内置的设备证书 [公钥](https://en.wikipedia.org/wiki/Public-key_cryptography)) ，创建 AWS IoT 事物（您的设备的表示形式和记录）。安全元件唯一的序列号将会作为 client Id，client Id 用于在 AWS IoT Core 中注册和识别设备。您可以使用类似流程，一次自动执行数千或数百万台设备的批量部署。

## 打开 Blink Hello World 项目
如果你有任何其他项目已经在 VS Code 中打开，首先打开一个新窗口 (**File** → **new window**) 使用一个干净的文件浏览器和工作环境。

对于本教程，您将使用 Blinky-Hello-World 项目。在你的新 VS Code 窗口中，VS Code 活动栏(最左边的菜单)内，点击 **PlatformIO 徽标**，从左边的 PlatformIO 菜单选择 **Open**，然后点击 **Open Project**，导航到 `Core2-for-AWS-IoT-EduKit/Blinky-Hello-World` 文件夹，并点击 **Open**。
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - 打开 PIO 菜单，2 - 打开 PIO home，3 - 打开项目文件夹" >}}

## 检索设备证书并注册 AWS IoT 事物 (thing)
现在创建一个基于 [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) 的安全的 TLS 连接以连接到 AWS IoT Core，您需要注册一个事物 (thing)，在事物上 [附加设备证书](https://docs.aws.amazon.com/iot/latest/developerguide/register-device-cert.html)（公钥），然后在证书上附加一个 [安全策略](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) 以确保非法设备或非法操作不会在您的AWS帐户内执行。通过在 Core2 for AWS IoT EduKit 参考硬件包含的安全元件，我们可以无需暴露或处理敏感的私钥自动完成整个设备注册。项目中包含一个脚本，该脚本可以实现流程自动化；它从参考硬件的安全元件中检索预先准备的设备证书，通过使用 [X.509 证书](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html#x509-client-cert-basics) 对设备证书签名来生成设备清单，执行 Microchip(安全元件制造商) CA 的[just-in-time registration（即时注册）](https://aws.amazon.com/blogs/iot/just-in-time-registration-of-device-certificates-on-aws-iot/)，在 AWS IoT 中使用设备证书注册事物，并将安全策略附加到 AWS IoT 事物上。

使用 [PlatformIO CLI 终端窗口](prerequisites.html#open-the-platformio-cli-terminal-window) 运行脚本，如下可以根据您的操作系统展开相应的说明：
{{%expand "Ubuntu or macOS" %}}
要运行注册脚本（位于 **utilities\AWS_IoT_registration_helper** 目录中），请将下面的命令复制并粘贴到 PlatformIO CLI 终端窗口中。在运行它之前，将 **<<DEVICE_PORT>>** 替换为设备实际挂载的端口。要再次查找端口，请参阅 [Ubuntu](../getting-started/prerequisites/linux.html#identifying-the-device-communication-port) 或者 [macOS](../getting-started/prerequisites/macos.html#identifying-the-device-communication-port) 的说明。
```bash
cd utilities/AWS_IoT_registration_helper
python3 registration_helper.py -p <<DEVICE_PORT>>
```

在 AWS IoT 中成功注册和预置设备后，返回到 **Blinky-Hello-World** 目录中：
```bash
cd ../..
```
{{% /expand%}}
{{%expand "Windows" %}}
要运行注册脚本（位于 **utilities\AWS_IoT_registration_helper** 目录中），请将下面的命令复制并粘贴到 PlatformIO CLI 终端窗口中。在运行它之前，将 **<<DEVICE_PORT>>** 替换为设备实际挂载的端口。要再次查找端口，请参阅 [Windows](../getting-started/prerequisites/windows.html#identifying-the-device-communication-port)。
```PowerShell
cd utilities\AWS_IoT_registration_helper\
python registration_helper.py -p <<DEVICE_PORT>>
```

在 AWS IoT 中成功注册和预置设备后，返回到 **Blinky-Hello-World** 目录中：
```PowerShell
cd ..\..
```
{{% /expand%}}

## 章节总结
在本章节中，您使用安全元件和注册脚本创建了 AWS IoT [事物](https://docs.aws.amazon.com/iot/latest/developerguide/thing-registry.html)、为事物设置了权限 [策略](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html)，并向其附加了设备证书。所有这些操作都是在未暴露私有密钥和消除泄露密钥潜在威胁的情况下完成的。

进入到 [**连接到 AWS IoT Core**](connecting-to-aws.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}
