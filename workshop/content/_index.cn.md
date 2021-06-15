+++
title = "AWS IoT EduKit"
chapter = true
weight = 1
+++
![EduKit Logo](AWS_IoT_EduKIt_Logo-320px_193px.png)
AWS IoT EduKit 是用于通过规范性学习计划，构建使用 AWS 服务的 IoT 应用程序的简单途径。无论是学生还是经验丰富的工程师和专业人员，AWS IoT EduKit 都可以通过将参考硬件工具包和一套便于实施的指南和示例代码结合起来，帮助开发人员获得构建端到端 IoT 应用程序的实践经验。

### AWS IoT EduKit 的优势
#### 简化硬件的选择
- AWS IoT EduKit 参考硬件工具包由我们的制造合作伙伴 M5Stack 生产并销售，它具备多项开箱即用的板载功能，可支持很多 IoT 应用程序。开发人员可以利用即插即用的扩展选项，将其应用于更多使用案例。
- 参考硬件工具包提供功能强大且安全的硬件，支持以入门级价格构建各种 IoT 应用程序（从入门级到高级/专业级）。它由 ESP32 MCU 和 Microchip ATECC608 Trust&GO 安全元件提供支持。它具备 Alexa 功能，配备了多个板载外围设备、各种可单独使用的扩展模块可以用于其他连接选项（例如 LoRaWAN、NB-IoT、LTS），以及即插即用的 Grove 连接器外围设备（例如传感器、执行器）可以应用于更多使用案例。

#### 支持广泛的软件框架
- AWS IoT EduKit 参考硬件支持各种应用程序框架（例如 FreeRTOS、Arduino、MicroPython），这使得开发人员能够使用他们所擅长的语言来编写代码，并在单个硬件平台上构建与云连接的嵌入式应用程序。
- AWS IoT EduKit 可以通过 Espressif 的 Rainmaker 平台和 PlatformIO 开发平台简化 AWS IoT 入门流程。Espressif Rainmaker 平台让您能够在没有 AWS 账户的情况下，控制参考硬件工具包和智能家居应用程序。PlatformIO 开发平台简化了嵌入式开发，从而可以更快地浏览、编辑和烧录代码。

#### 轻松访问示例代码
- AWS IoT EduKit 为大部分常见的 IoT 解决方案提供最新内容和示例代码。
- 通过 AWS IoT EduKit [教程](https://edukit.workshop.aws/cn/getting-started.html)，开发人员可以访问各种免费内容，以获取使用 AWS 服务构建和管理 IoT 应用程序的专业知识。

### 工作原理
要开始使用，请购买我们的制造合作伙伴 M5Stack 的参考硬件工具包，可以从 [Amazon.com](https://www.amazon.com/dp/B08VGRZYJR/) (US) 购买，也可以直接从[M5Stack 店铺](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) (CN/Global) ，[Digi-Key](https://www.digikey.com/en/products/detail/m5stack-technology-co-ltd/K010-AWS/13562927) (AMER/Global)，[Switch Science](https://www.switch-science.com/catalog/6784/) (JP)，或者 [Distrelec](https://www.distrelec.biz/en/esp32-m5core2-iot-development-kit-for-aws-iot-edukit-m5stack-k010-aws/p/30196462) (EU) 等处购买。

获取硬件工具包之后，请查看 [入门教程](https://edukit.workshop.aws/cn/getting-started.html) 并按照相关步骤在手机上安装 Espressif Rainmaker 移动应用程序和 RainMaker 代理固件。借助 Espressif Rainmaker 移动应用程序，您可以控制 AWS IoT EduKit 参考硬件并连接到 AWS IoT。接下来，您可以从 AWS IoT EduKit 网站上的可用免费项目列表中进行选择。从构建基本的互联家居应用程序开始，并随着时间的推移，逐步发展到使用 Amazon SageMaker Autopilot 运行机器学习模型，或使用 Alexa Voice Service Integration for AWS IoT (AFI) 构建语音辅助智能家居应用程序。要详细了解硬件规范，请访问 [M5Stack 文档](https://docs.m5stack.com/#/en/core/core2_for_aws)  或 [Partner Device Catalog](https://devices.amazonaws.com/detail/a3G0h000007djMLEAY)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}