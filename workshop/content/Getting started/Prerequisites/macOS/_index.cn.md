+++
pre = "› "
linkTitle = "macOS"
title = "MacOS 10.14+ 配置步骤"
+++

本部分内容展示了怎样配置您的 macOS 计算机（主机），完成之后，您将可以从 github 仓库中下载代码，查看和编辑代码，并进行编译，然后写入到硬件的闪存中去。这些安装步骤使用了 Espressif 的用于 RainMaker 平台的 AWS 账号和服务，以满足 **入门** 演练的需求。

## 安装 Silicon Labs USB to UART Bridge 驱动程序
Core2 for AWS IoT EduKit 通过一个 Silicon Labs CP210x USB to UART bridge 和主机通讯。板载的 CP2104 是一个 USB-to-UART bridge 用于实现和 ESP32-D0WD 微控制器之间的主机通讯。微控制器通过 [UART](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/uart.html)0（CP210x 在主机上通过 USB-C 创建的一个虚拟端口） 进行双向通讯。为了挂载虚拟串口并通过该串口进行通讯，您需要下载安装相应的驱动程序。

1) 确保设备没有连接到主机上。
2) 下载 macOS Silicon Labs CP210x 驱动程序 [这里](https://www.silabs.com/documents/public/software/Mac_OSX_VCP_Driver.zip)。
3) 解压下载的文件，并挂载 **SiLabsUSBDriverDisk.dmg** 磁盘镜像。
4) 运行 **Install C210x VCP Driver**。
   - 在 macOS 10.13 及以上的版本，可能会阻止 SiLabs system extension 的安装。要允许安装需要进行下面的操作，打开您的 Mac 中的 **系统偏好设置** <i class="fas fa-arrow-right"></i> **安全性与隐私** 面板, 点击 <i class="fas fa-lock"></i>, **点按锁按钮以进行更改** 解锁并进行更改，然后通过点击 <i class="fas fa-lock-open"></i>重新锁定。更多信息，请参考 [Apple Technical Note TN2459](https://developer.apple.com/library/archive/technotes/tn2459/_index.html)。
5) 重启您的主机，并确认驱动程序已经安装完毕。

## 安装 Visual Studio Code
Visual Studio Code (VS Code) 是一个开源集成式开发环境 (IDE)，让您可以查看、编辑和管理代码等。下载适用于您的操作系统的最新版本 [Visual Studio Code](https://code.visualstudio.com/)。要排查 Visual Studio Code 安装或使用方面的问题，请参阅[相关文档](https://code.visualstudio.com/docs/setup/setup-overview)。

{{% notice note %}}
该应用程序位于您的 **Downloads** 文件夹中。我们建议您选择将其移动到用户的 **Application** 文件夹中，以便将来更容易访问。
{{% /notice %}}

## 安装 PlatformIO
[PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) (PIO)  提供了专业的嵌入式开发平台，该平台可以简化嵌入式软件
开发过程。Visual Studio Code extension 在图形界面中提供了 PlatformIO 命令行界面 (CLI) 的功能。您可以在[此处](https://platformio.org/install/ide?install=vscode)下载 extension 并阅读有关 PlatformIO 的更多信息。

在 PlatformIO extension 安装完毕之后，您需要重启 VS Code。

## 克隆代码仓库
所有的项目和文件都托管在 [GitHub 仓库](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/about-repositories) 中，您还可以在代码仓库（repo）中查看每一个文件的历史版本。克隆演练中所需要的代码，您会使用 PIO 接口：
1) 在 VS Code 活动栏（最左侧菜单）中点击 PlatformIO 图标。
2) 从 PlatformIO 的 **Quick Access** 菜单里，在 **Miscellaneous** 的下方，选择 **Clone Git Project**。
3) 复制 `https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git` 到图中所示区域，并选择项目保存的路径。
{{< img "pio-clone_git_project.en.png" "PlatformIO Clone Git Project" "1 - 打开 PIO 菜单， 2 - 克隆 git 项目， 3 - 复制仓库 URL" >}}

## 下载并安装手机 apps
ESP RainMaker 手机应用程序适用于 iOS 和 Android 手机，可提供 Wi-Fi 网络配置、用户创建、用户设备关联和设备控制。这些应用程序可以在以下位置找到：
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker), [Direct APK](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

如果您没有兼容的 Android 或 iOS 设备，则可以使用 [RainMaker CLI](https://rainmaker.espressif.com/docs/cli-setup.html) 和替代指令。

## 确认设备通讯端口
现在可以拿出您的 Core2 for AWS IoT EduKit 参考硬件，使用 USB-A to USB-C 线缆连接到主机的 USB 2.0 接口上。此外，包装内的六角扳手可以用于安装其他的附加模组（需单独购买）。现在设备应该在您插上时自动开启了，但如果需要手动开启，请按下电源按键。
![How to turn M5Stack Core2 for AWS on or off](macos/core2foraws_power_on_off.jpg?width=500px&classes=shadow)

至此，设备已经就绪，并且演练所需软件也已经安装完毕。让我们确认您的设备虚拟挂载的端口，之后使用该端口进行读写操作。
1) 从 PlatformIO 的 **Quick Access** 菜单中， 在 **PIO Home** 下面， 选择 **Devices**。
2) 查看有描述为 "CP2104 USB to UART Bridge Controller" 的端口（通常是/dev/cu.SLAB_USBtoUART），并点击旁边的图标复制设备端口。

{{% notice note %}}
如果您的 Core2 for AWS IoT EduKit 并没有在设备列表中显示，检查设备是否开机以及您正在使用的 USB-A to USB-C 线缆。某些 USB-C hubs 在创建串口时可能会有兼容性问题。
{{% /notice %}}

## 接下来
所有配置已经完成，您的主机也已经就绪并能够和 Core2 for AWS IoT EduKit 参考硬件进行通信，我们继续进行下一章的内容 — [**运行 ESP RainMaker 代理**](/cn/getting-started/run-rainmaker.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}
