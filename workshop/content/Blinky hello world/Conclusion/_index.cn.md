+++
title = "总结"
weight = 50
pre = "<b>e. </b>"
+++

希望您喜欢构建连接到云的 Blinky 项目的过程。您已经使用 PlatformIO 配置、构建、烧录和监控了 Core2 for AWS IoT EduKit 的固件。现在，您的设备已经在您的 AWS 账户中注册为 AWS IoT 事物，使用永远不会离开设备的嵌入式设备证书和私有密钥来实现安全的云连接。您连接到了 AWS IoT Core，通过 MQTT 从设备发送了消息，并从 AWS IoT 控制台的 MQTT 客户端接收了 MQTT 消息，并可以从 MQTT 客户端切换设备上的指示灯。

虽然 AWS IoT EduKit 项目不教授 C 语言，也不要求对 C 语言的了解，但它对嵌入式开发至关重要。来自底层代码的控制和灵活性允许您从硬件中获得最大的潜力。如果你不熟悉 C 编程，但想学习或者你希望深入我们提供的应用程序代码，Jens Gustedt 在 Creative Commons license 下提供了 [Modern C](https://gustedt.gitlabpages.inria.fr/modern-c/)，它可以免费 [下载](https://gustedt.gitlabpages.inria.fr/modern-c/#orge4fc44a)。

## 设备上发生了什么
## FreeRTOS 内核
设备上的应用程序使用 [FreeRTOS 内核](https://www.freertos.org/) - [一种实时操作系统](https://www.freertos.org/about-RTOS.html) (RTOS)。它绑定到了您应用程序代码编译的二进制文件中（以及其他库和驱动程序）。FreeRTOS 内核将一个代码块作为一个 [task](https://www.freertos.org/taskandcr.html) 进行调度，以共享处理器运行时。有些任务通常会永远循环，以创建要发送到 AWS 的消息、处理接收到消息、更新显示、闪烁 LED 等。

Core2 for AWS IoT EduKit 封装了一个 240MHz 双核处理器，然而，与目前常见的 4+GHz 的 8+ 核计算机相比，它显得很无力。为了充分利用宝贵的处理器时间，FreeRTOS 内核将 [tasks in a states（任务置于一种状态）](https://www.freertos.org/RTOS-task-states.html)，例如运行或挂起。FreeRTOS 内核为微控制器应用程序提供了通过快速切换单个任务来优化处理器的能力。一个任务可以在另一个任务处于挂起或阻塞状态时运行（见下图）。与更简单的 [super loop applications](https://en.wikibooks.org/wiki/Embedded_Systems/Super_Loop_Architecture) 相比，这提供了并发的表象，并极大地提高了嵌入式应用程序的性能。
{{< img "RTOS_scheduling_execution.en.png" "Example of FreeRTOS application scheduling" "1 - MQTT 开始运行，2 - MQTT 阻塞 (没有新消息) & 显示开始运行，3 - MQTT 运行，处理消息 & 显示阻塞，4 - MQTT 阻塞 & 闪烁任务运行 (LED 开启)，5 - 闪烁阻塞 & 显示运行, 6 - 显示阻塞 & 闪烁运行 (LED 关闭)，7 - 闪烁阻塞 & 显示运行">}}

虽然 super loop applications 更容易编程，但代码的区域阻塞/延迟其他代码的执行导致 MCU 处于闲置状态。连接到 AWS 云的设备将有变化的往返延迟和带宽限制，这使得使用 RTOS 的性能优势成为必要。RTOS 为其他应用程序提供了显著的优势，在这些应用程序中，最重要的是不丢失数据，能够执行关键操作而不被其他代码块的执行（例如健康和安全）所阻碍，或者仅仅是驱动一个显示屏。
{{< img "super_loop_execution.en.png" "Example of super loop application execution" "1 - MQTT 终端开始，2 - MQTT 消息处理完成 & LED 切换到开，代码执行暂停 500ms，LED 切换到关，暂停执行 500ms，3 - 闪烁代码结束 & 开始显示刷新，4 - 显示代码结束 & LED 切换为开，暂停执行 500ms，LED 关，暂停执行 500ms" >}}

要学习更多关于 RTOS 和 FreeRTOS 的内容，查看 [这些视频](https://www.youtube.com/watch?v=F321087yYy4) 并浏览 [FreeRTOS.org](https://www.freertos.org/RTOS.html)。

## AWS IoT Device SDK for Embedded C
在本示例中使用的 [AWS IoT Device SDK for Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) 简化了与 AWS IoT 的连接和消息 收/发，并能够和安全元件共同工作简化设备认证。Device SDK for Embedded C 在 `Core2-for-AWS-IoT-EduKit/Blink-Hello-World/components/esp-aws-iot/` 文件夹内。

## 板级支持包 — 设备驱动程序
设备的硬件功能，如屏幕、电源管理芯片、安全元件、扬声器、麦克风、六轴 [惯性测量单元](https://en.wikipedia.org/wiki/Inertial_measurement_unit) (IMU)、触摸驱动器、LED条等都捆绑为一个板级支持包 (BSP)。BSP 是带有 APIs 的驱动程序，提供了一种访问硬件特性的简化方式。BSP 位于 `Core2-for-AWS-IoT-EduKit/Blink-Hello-World/components/core2foraws/` 文件夹中。在本例中，SK6812 驱动程序用于控制侧 LED 条，[LVGL库](https://docs.lvgl.io/v7/en/html/) 用于在显示器上显示元素。要查看 BSP 可用的 API (application programming interface, API) 及其使用，请参见 <a href="https://edukit.workshop.aws/en/api-reference/index.html" target="_blank">Core2 for AWS IoT EduKit API 参考</a>。

进入下一章节，[**智能恒温器**](/cn/smart-thermostat.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}