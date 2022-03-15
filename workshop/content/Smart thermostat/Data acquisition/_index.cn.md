+++
title = "数据获取"
weight = 20
pre = "<b>b. </b>"
+++

## 章节简介
学习完本章节后，您的设备将能够：

* 开机并启动本地智能恒温器应用程序。
* 将检测到的当前温度、环境声音强度、HVAC 系统状态（制热/制冷/待机）和空间占用指标报告到日志中。

Core2 for AWS IoT EduKit 参考硬件套件自带几个可以随时使用的传感器。针对这一解决方案，您需要使用本地应用程序代码从温度传感器和麦克风读取数据。应用程序将跟踪传感器读数和状态标志，以便在显示器中提供汇总数据。

## 如何对恒温器应用程序进行编程
您的智能恒温器将使用已经创建并且包含在捆绑软件组件中的代码来对集成传感器进行采样。在第一步中，您只需要获取相关值并将其打印到日志记录中，后续我们会将传感器值发布到 AWS IoT Core。

您可以很容易地从附带的 MPU6886 模块的温度传感器读取数值。以下是代码片段：

```c
// include libraries for interfacing with the kit's modules
#include "freertos/FreeRTOS.h"
#include "core2forAWS.h"
#include "mpu6886.h"
#include "esp_log.h"

// the application to run on the device
void app_main()
{
  // initialize a float to store our temperature reading
  float temperature = 0.0f;

  // initialize the kit modules
  Core2ForAWS_Init();
  MPU6886_Init();

  // read from the temperature sensor
  MPU6886_GetTempData(&temperature);

  // convert the reading to Fahrenheit and apply a calibration offset of -50
  temperature = (temperature * 1.8)  + 32 - 50;

  // write the value to the ESP32 logger
  ESP_LOGI("thermostat", "measured temperature is: %f", temperature);
}
```

如果您要使用这段代码来构建、烧录和监控设备日志，那么设备会从温度传感器读取一个读数并将其写入日志记录，然后停止。要在不阻塞其他进程的前提下实现每秒连续采样，您要创建单独的 **[FreeRTOS task](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/system/freertos.html#_CPPv423xTaskCreatePinnedToCore14TaskFunction_tPCKcK8uint32_tPCv11UBaseType_tPC12TaskHandle_tK10BaseType_t)** 任务并将其固定到 MCU 核心，如下所示：

```c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "core2forAWS.h"
#include "mpu6886.h"
#include "esp_log.h"

// store our application logic in a task function
void temperature_task(void *arg) {
  float temperature = 0.0f;

  // loop forever!
  for (;;) {
    MPU6886_GetTempData(&temperature);
    temperature = (temperature * 1.8)  + 32 - 50;
    ESP_LOGI("thermostat", "measured temperature is: %f", temperature);

    // sleep for 1000ms before continuing the loop
    vTaskDelay(1000 / portTICK_RATE_MS);
  }
}

void app_main()
{
  Core2ForAWS_Init();
  MPU6886_Init();

  // FreeRTOS concept: operations that run in a continuous loop are done in tasks
  xTaskCreatePinnedToCore(&temperature_task, "temperature_task", 4096, NULL, 5, NULL, 1);
}
```

## 编译并上传烧写智能恒温器固件
已经为您准备了一个示例应用程序，您可以使用 Visual Studio Code 和 PlatformIO extension 构建并上传到您的设备。如果你有任何其他项目已经在 VS Code 中打开，首先打开一个新窗口（文件 → 新窗口）以使用一个干净的文件资源管理器和工作环境。

对于本教程，您将使用智能恒温器项目。在您的新的 VS Code 窗口中： 
1. 在 VS Code 活动栏（最左侧菜单），点击 **PlatformIO 徽标**。
2. 从左侧的 **PlatformIO 菜单**中，选择 **Open**。
3. 点击 **Open Project**。
4. 浏览到 `Core2-for-AWS-IoT-EduKit/Smart-Thermostat` 文件夹，然后点击 **open**。
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - 打开 PIO 菜单，2 - 打开 PIO home，3 - 打开项目文件夹" >}}

接下来，您需要在 VS Code 中打开一个新的 PlatformIO CLI 终端窗口：
1) 在 VS Code 活动栏（最左侧菜单），点击 **PlatformIO 徽标**。
2) 从 **Quick Access** 菜单中，在 **Miscellaneous** 的下方，选择 **New Terminal**。

{{< img "pio-new_terminal-smart_thermostat.en.png" "PlatformIO CLI terminal in VS Code" "1 - 打开 PIO 菜单，2 - 打开新的 PIO 终端，3 - 确认您处于 'PlatformIO CLI' 终端会话，4 - 复制命令到终端中，5 - 如果在自动探测端口时您遇到一个错误，打开 Platform.ini 文件并参照手动添加端口的步骤。">}}


 从 Blinky 项目中复制配置，编译设备固件，以及上传到您的设备中去，根据您的主机操作系统，参照下面的步骤完成本章节：

{{%expand "Ubuntu or macOS" %}}
1. 从 **Blinky-Hello-World** 教程中复制您的 `sdkconfig` 文件，这样可以简化 Wi-Fi 和 AWS endpoint 的配置：
   ```bash
   cp ../Blinky-Hello-World/sdkconfig .
   ```
   {{% notice note %}}
   作为可选，您可以手动为您的固件设置 Wi-Fi 凭证和 AWS 端点，操作说明在 **Blinky Hello World** 示例中的 **[配置 ESP32 固件](/cn/blinky-hello-world/connecting-to-aws.html#configuring-the-esp32-firmware)** 步骤中。
   {{% /notice %}}
2. 使用下面的命令来编译和上传固件，并监视设备的串行输出。这将花费一些时间以构建和烧写应用程序，但在那之后，你应该在您的终端中看到设备日志流。你可以用 **Ctrl** + **C** 组合键关闭监视会话:
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```
{{% /expand%}}
{{%expand "Windows" %}}
1. 从 **Blinky-Hello-World** 演示中复制您的 `sdkconfig` 文件，这样可以简化 Wi-Fi 和 AWS endpoint 的配置：
   ```bash
   copy ..\Blinky-Hello-World\sdkconfig .
   ```
   {{% notice note %}}
   作为可选，您可以手动为您的固件设置 Wi-Fi 凭证和 AWS 端点，操作说明在 **Blinky Hello World** 示例中的 **[配置 ESP32 固件](/cn/blinky-hello-world/connecting-to-aws.html#configuring-the-esp32-firmware)** 步骤中。
   {{% /notice %}}
2. 使用下面的命令来编译和上传固件，并监视设备的串行输出。这将花费一些时间以构建和烧写应用程序，但在那之后，你应该在您的终端中看到设备日志流。你可以用 **Ctrl** + **C** 组合键关闭监视会话:
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```
{{% /expand%}}

## 检验
在进入下一章节之前，您可以通过查看终端窗口中的串行输出来验证你的设备是否按照预期进行了配置：

```
I (16128) MAIN: On Device: roomOccupancy false
I (16132) MAIN: On Device: hvacStatus STANDBY
I (16137) MAIN: On Device: temperature 64.057533
I (16143) MAIN: On Device: sound 8
```

如果符合预期，我们就进入到 [**数据同步**](/cn/smart-thermostat/data-sync.html) 部分。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}