+++
title = "运行 ESP RainMaker 代理"
weight = 12
pre = "<b>b. </b>"
+++

我们已经准备好了编译应用程序并将其烧录到设备，通过 ESP Rainmaker 手机应用程序预配置 Wi-Fi，注册用户以将设备分配给该用户，以及开始控制板载外围设备将其作为基于 AWS IoT 的虚拟智能家居设备。

## 在 PlatformIO 中打开项目
在您之前从 GitHub 克隆的项目的根目录下有几个文件夹。在本教程中，您将使用入门项目并进行 PlatformIO 中所需的操作。首先打开 Visual Studio Code，等待几秒钟加载 PlatformIO extension，单击 VS Code 活动栏（左侧菜单）中的 **PlatformIO 图标**，从 PlatformIO 菜单中选择 **Open（打开）**，点击 **Open Project（打开项目）** 导航到 `Core2-for-AWS-IoT-EduKit/Getting-Started` 文件夹，最后点击 **Open（打开）**。
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - 打开 PIO 菜单，2 - 打开 PIO Home，3 - 打开 Getting Started 项目" >}}

## 编译和上传 RainMaker 代理固件
现在，您已经准备好构建（编译）和上传 RainMaker 代理固件了。使用 [GCC 编译器](https://gcc.gnu.org/onlinedocs/gcc/) 完成编译操作，编译操作将人类可读的代码转换为 [elf](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) 和二进制文件（固件）形式的 [对象码](https://en.wikipedia.org/wiki/Object_code)。这些二进制文件后续会通过虚拟串口上传烧写到设备板载闪存中去进行执行。虚拟串口（通过 UART）允许双向通信，所以，你也能从设备到主机接收数据。为了构建和上传固件，为了使用 PlatformIO 通过虚拟串口监控设备的输出，需要进行如下操作：
1) 在 VS Code 活动栏（左侧菜单）中，点击 **PlatformIO 图标**。
2) 从 **Quick Access** 菜单中，在 **Miscellaneous** 下方，选择 **New Terminal**。
3) 要开始构建过程，在一个新打开的 VS Code 终端中输入下面的命令：
    ```bash
    pio run --environment core2foraws
    ```

{{% notice info %}}
PIO 会为设备平台安装一些必要的依赖项。如果这些操作不完整，则可能会遇到错误。等待一分钟或两分钟并重新运行上面的命令应该可以解决这个问题。
{{% /notice %}}

{{< img "pio-new_terminal-gs.en.png" "PlatformIO CLI terminal in VS Code" "1 - 打开 PIO 菜单，2 - 打开新的 PIO 终端，3 - 核实您正在使用 `PlatformIO CLI` 终端会话，4 - 将命令复制到终端中，5 - 如果在自动检测端口时遇到错误，请打开 Platform.ini 文件并按照说明手动添加串口。">}}

现在，可以开始通过 USB 接口将编译完成的固件上传到连接到设备中，同时执行命令来监视通过串口输出到主机上的信息
   ```bash
   pio run --environment core2foraws --target upload --target monitor
   ```
{{% notice info %}}
如果在上传或监控串行输出过程中收到关于错误端口或超时的错误，请打开 `platformio.ini` 文件，并按照该文件中的说明手动设置上传端口。
{{% /notice %}}
## 声明和预置设备
一旦上传成功完成，设备将用刚刚编译和上传的固件启动。它还将在终端视图中显示该设备的串行输出。该设备将生成安全密钥和执行 [assisted claim](https://rainmaker.espressif.com/docs/claiming.html#assisted-claiming-esp32) 的过程。密钥生成可能需要几秒钟最多几分钟才能完成，但一旦申领完成，一个二维码将显示在终端视图。

打开手机上的 ESP RainMaker 手机 App，授予其请求的应用程序权限，按下 **Add Device（添加设备）**，然后扫描计算机串行监控器终端视图上显示的二维码。随后将开始预置过程，其中包括使用 Wi-Fi 凭证为 2.4GHz 无线家庭网络预置 Wi-Fi。成功连接 Wi-Fi 后，设备将对自身进行身份验证。手机 App 将装载多个可查看和/或控制的虚拟设备。如果一两分钟后手机 App 上的虚拟设备显示为 **offline(离线)**，则向下滚动进行刷新。
{{< img "pio-qr_code_scan.en.png" "在串行输出中扫码二维码" >}}

在虚拟设备显示在移动应用程序中并处于在线状态后，您可以打开或关闭板载电机或 LED，调整电机的速度，设置 LED 灯条的颜色和亮度，以及查看设备内部温度。

{{% notice info %}}
如果您输入的 Wi-Fi 凭证错误，则需要先 [擦除固件](/cn/getting-started/run-rainmaker.html#erasing-the-firmware-with-platformio)，将 Espressif RainMaker 代理固件重新上传到设备，并再次使用手机添加设备。有关更多问题排查或常见问题，请访问官方 [Espressif RainMaker 常见问题](https://rainmaker.espressif.com/docs/faqs.html)。
{{% /notice %}}

## 使用 PlatformIO 擦除固件
学完本教程并准备学习其他教程时，您首先需要擦除设备固件。要执行此操作，您需要使用 **CTRL** + **C** 组合键停用活动的串行监视器。随后输入如下命令擦除固件：
```bash
pio run --environment core2foraws --target erase
```

{{% notice note %}}
设备的闪存为空的时候，将导致设备不断重启且设备屏幕一片空白，并从扬声器发出滴答声。这是预期的行为，因为设备在没有可以运行应用程序的情况下会不断地重新启动自身。
{{% /notice %}}

## 总结
您刚刚通过 AWS IoT EduKit 构建了一个互联家居应用程序！不仅如此，您还拥有必要的工具来创建、编辑和编译嵌入式代码并将其烧录到设备上。 在后续教程中，您将获得更多动手实践机会并学习相关技能，来构建自己的端到端 IoT 解决方案。

进入到 [**Blinky Hello World（连接到云的 Blinky）**](/cn/blinky-hello-world.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}
