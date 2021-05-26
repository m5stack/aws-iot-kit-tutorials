+++
pre = "› "
linkTitle = "Linux"
title = "Ubuntu Linux v18.0+ 配置步骤"
+++

本部分内容展示了怎样配置您的 linux 计算机（主机），完成之后，您将可以从 github 仓库中下载代码，查看和编辑代码，并进行编译，然后烧录到硬件的闪存中去。这些安装步骤使用了 Espressif 的用于 RainMaker 平台的 AWS 账号和服务，以满足 **入门** 演练的需求。

## 安装依赖包
为了下载代码，[编译代码](https://en.wikipedia.org/wiki/Object_code)，以及运行脚本使用 [Python 包管理器](https://pip.pypa.io/en/stable/)下载后续的依赖包，您需要先安装一些依赖库和应用程序。要从远程 GitHub 代码仓库下载代码，您需要安装 [git](https://git-scm.com/)。为了编译代码，Espressif 使用 [cmake](https://cmake.org/) 构建系统解析设备应用程序的配置参数，使用 [gcc compiler](https://gcc.gnu.org/onlinedocs/gcc/) 编译成 Core2 for AWS IoT EduKit 可以理解和运行的目标代码。所有的依赖包可以简单的在终端种使用 apt 安装：

```bash
sudo apt install build-essential python3-pip curl git cmake
```

## 安装 Silicon Labs USB to UART Bridge 驱动程序
Core2 for AWS IoT EduKit 通过一个 Silicon Labs CP210x USB to UART bridge 和主机通讯。板载的 CP2104 是一个 USB-to-UART bridge 用于实现和 ESP32-D0WD 微控制器之间的主机通讯。微控制器通过 [UART](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/uart.html)0（CP210x 在主机上通过 USB-C 创建的一个虚拟端口） 进行双向通讯。

Linux 内核版本 3.x.x and 4.x.x 已经包含了 CP210x 所需的驱动程序。然而，在默认情况下，串口通讯会被限制为 dialout 群组中的用户。为了将您的用户加入到 dialout 群组中，在终端中输入如下命令：

```bash
sudo usermod -a -G dialout $USER
```

重启您的主机，以使得配置生效。

## Visual Studio Code 安装
Visual Studio Code 是一个开源集成式开发环境 (IDE)，让您可以查看、编辑和管理代码等。下载适用于您的操作系统的最新版本 [Visual Studio Code](https://code.visualstudio.com/)。要排查 Visual Studio Code 安装或使用方面的问题，请参阅 [相关文档](https://code.visualstudio.com/docs/setup/setup-overview)。

## 安装 PlatformIO
[PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) (PIO)  提供了专业的嵌入式开发平台，该平台可以简化嵌入式软件开发过程。Visual Studio Code extension 在图形界面中提供了 PlatformIO 命令行界面 (CLI) 的功能。您可以在 [此处](https://platformio.org/install/ide?install=vscode) 下载 extension 并阅读有关 PlatformIO 的更多信息。

在 PlatformIO extension 安装完毕之后，您需要重启 VS Code。 

## 克隆代码仓库
所有的项目和文件都托管在 [GitHub 仓库](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/about-repositories) 中，您还可以在代码仓库（repo）中查看每一个文件的历史版本。克隆演练中所需要的代码，您会使用 PIO 接口：
1) 在 VS Code 活动栏（最左侧菜单）中点击 PlatformIO 图标。
2) 从 PlatformIO 的 **Quick Access** 菜单里，在 **Miscellaneous** 的下方，选择 **Clone Git Project**。
3) 复制 `https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git` 到图中所示区域，并选择项目保存的路径。
{{< img "pio-clone_git_project.en.png" "PlatformIO 克隆 Git 项目" "1 - 打开 PIO 菜单， 2 - 克隆 git 项目， 3 - 复制仓库 URL" >}}

## 下载并安装手机应用程序
ESP RainMaker 手机应用程序适用于 iOS 和 Android 手机，可提供 Wi-Fi 网络配置、用户创建、用户设备关联和设备控制。这些应用程序可以在以下位置找到：
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker), [Direct APK](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

如果您没有兼容的 Android 或 iOS 设备，则可以使用 [RainMaker CLI](https://rainmaker.espressif.com/docs/cli-setup.html) 和替代指令。

## 确认设备通讯端口
现在可以拿出您的 Core2 for AWS IoT EduKit 参考硬件，使用 USB-A to USB-C 线缆连接到主机的 USB 2.0 接口上。此外，包装内的六角扳手可以用于安装其他的附加模组（需单独购买）。现在设备应该在您插上时自动开启了，但如果需要手动开启，请按下电源按键。
![如何开关 M5Stack Core2 for AWS](linux/core2foraws_power_on_off.jpg?width=500px&classes=shadow)

设备已经就绪，并且演练所需软件也已经安装完毕。让我们确认您的设备虚拟挂载的端口，之后使用该端口进行读写操作。
1) 从 PlatformIO 的 **Quick Access** 菜单中， 在 **PIO Home** 下面， 选择 **Devices**。
2) 查看有描述为 "CP2104 USB to UART Bridge Controller" 的端口（通常是/dev/ttyUSB0），并点击旁边的图标复制设备端口。

{{% notice note %}}
如果您的 Core2 for AWS IoT EduKit 并没有在设备列表中显示，检查是否开机以及您正在使用的 USB-A to USB-C 线缆。某些 USB-C hubs 在创建串口时可能会有兼容性问题。
{{% /notice %}}

## 接下来
所有配置已经完成，您的主机也已经就绪并能够和 AWS IoT EduKit 参考硬件进行通信，我们继续进行下一章的内容 — [**运行 ESP RainMaker 代理**](/cn/getting-started/run-rainmaker.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}
