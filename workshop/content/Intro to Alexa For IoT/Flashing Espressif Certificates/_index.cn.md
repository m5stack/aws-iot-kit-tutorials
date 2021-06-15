+++
title = "烧录 Espressif 证书"
weight = 20
pre = "<b>b. </b>"
+++

## 章节简介
在本章节结束时，您将完成以下操作：在移动设备上安装配套手机应用程序，克隆必需的代码存储库，收到使用 Espressif Alexa 的 AWS 账户连接到 AWS IoT 所需的证书，以及将证书烧录到参考硬件的独立闪存分区中。

## 配套移动应用程序
为了完成与 Alexa 之间的身份验证，您需要 Espressif 的配套应用程序来为设备预置接入 WiFi，并预置 Alexa 账户。

下载 ESP Alexa 手机 App:
[iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

我们推荐 (对于本教程，可选) 您也安装了 Amazon Alexa app：[iOS](https://apps.apple.com/us/app/amazon-alexa/id944011620) 或者 [Android](https://play.google.com/store/apps/details?id=com.amazon.dee.app) - 这是和预置多数支持 Alexa 的设备时使用的同一个应用程序。

## 获取代码
本教程所用的所有代码都位于您在 [**Blinky Hello World（连接到云的Blinky）**](/cn/blinky-hello-world.html) 教程中克隆的存储库中的 `Alexa_for_IoT-Intro` 文件夹中。如果您已经克隆了该仓库，跳过本部分的操作。

如果需要从 [PlatformIO CLI 终端窗口](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window) 再次克隆该存储库，请运行以下命令：
```
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```

## 打开项目环境
对于此教程，您使用 Alexa_For_IoT-Intro 项目。在您的新的 VS Code 窗口中： 
1. 在 VS Code 活动栏（最左侧菜单）中，点击 **PlatformIO 徽标**。
2. 从左边的 PlatformIO 菜单中，选择 **Open**。
3. 点击 **Open Project**。
4. 导航到 `Core2-for-AWS-IoT-EduKit/Alexa_For_IoT-Intro` 文件夹，点击 **open**。
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - 打开 PIO 菜单，2 - 打开 PIO home，3 - 打开项目文件夹" >}}

接下来，您必须要在 VS Code 中打开一个新的 PlatformIO CLI 终端窗口：
1) 在 VS Code 活动栏（最左侧菜单）中，点击 **PlatformIO 徽标**。
2) 从 **Quick Access** 菜单中，在 **Miscellaneous**下方，选择 **New Terminal**。

{{< img "pio-new_terminal-smart_thermostat.en.png" "PlatformIO CLI terminal in VS Code" "1 - 打开 PIO 菜单，2 - 打开新的 PIO 终端，3 - 验证您正处于 'PlatformIO CLI' 终端会话中，4 - 将命令复制到终端内，5 - 如果在自动探测端口时遇到错误，打开 Platform.ini 文件并参考操作步骤手工添加端口。">}}

## 设置 AWS IoT 证书 
您需要创建 AWS IoT 证书，以便与 AWS IoT Core 进行通信。对于此研讨会和设备，Espressif 提供了可与 M5Stack Core2 for AWS IoT EduKit 参考硬件一起使用的 AWS IoT 证书。按照此处的步骤操作，获取您的证书以连接他们的服务。填写表格 [这里](https://espressif.github.io/esp-va-sdk/)。

您稍后会收到一封电子邮件，该邮件包含一个凭证压缩文件，保存该文件并解压。您将获得一个名为 **espcredentials** 的文件夹。插入设备，把这些证书上传到设备中，命令如下，您需要您的 PlatformIO CLI 终端窗口中进行操作。

{{%expand "Ubuntu or macOS" %}}
1. *除了* **mfg_config.csv** 文件以外，把 espcredentials 的内容复制到 `Core2-for-AWS-IoT-EduKit/Alexa_for_IoT-Intro/esp_alexa_credentials` 文件夹下。使用 **espcredentials** 文件夹的路径替换 **<<PATH_TO>>**。源代码的末尾 **/** 告诉 [rsync](https://download.samba.org/pub/rsync/rsync.1) 将内容复制到目标 `./esp_alexa_credentials/` 目录下，如果没有结尾的 **/** ，该操作将复制文件夹而非文件，并在下一步中导致文件未发现的错误：
   ```bash
   rsync -avr --exclude='mfg_config.csv' <<PATH_TO>>/espcredentials/ ./esp_alexa_credentials/
   ```
   
2. 使用下面的 PIO 命令运行脚本，该脚本将会使用凭证文件创建一个名为 `mfg.bin` 的二进制文件，该文件位于 esp_alexa_credentials 文件夹下，然后将二进制文件上传到板载的 [非易失性闪存存储](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/storage/nvs_flash.html) (NVS) 分区，后续应用程序将会将其作为 键：值 对进行访问：
   ```bash
   pio run --environment core2foraws --target flash_certificates
   ```
{{% /expand%}}
{{%expand "Windows" %}}
1. *除了* **mfg_config.csv** 文件以外，把 espcredentials 的内容复制到 `Core2-for-AWS-IoT-EduKit/Alexa_for_IoT-Intro/esp_alexa_credentials` 文件夹下。使用 **espcredentials** 文件夹的路径替换 **<<PATH_TO>>**。源代码的末尾 **/** 告诉 [rsync](https://download.samba.org/pub/rsync/rsync.1) 将内容复制到目标 `./esp_alexa_credentials/` 目录下，如果没有结尾的 **/** ，该操作将复制文件夹而非文件，并在下一步中导致文件未发现的错误：
   ```PowerShell
   robocopy "<<PATH_TO>>\espcredentials\" ".\esp_alexa_credentials\" /xf mfg_config.csv
   ```
   
2. 使用下面的 PIO 命令运行脚本，该脚本将会使用凭证文件创建一个名为 `mfg.bin` 的二进制文件，该文件位于 esp_alexa_credentials 文件夹下，然后将二进制文件上传到板载的 [非易失性闪存存储](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/storage/nvs_flash.html) (NVS) 分区，后续应用程序将会将其作为 键：值 对进行访问：
   ```PowerShell
   pio run --environment core2foraws --target flash_certificates
   ```
{{% /expand%}}

一切准备就绪后，我们来继续学习 [**构建和测试 AFI**](/cn/intro-to-alexa-for-iot/building-and-testing-afi.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}