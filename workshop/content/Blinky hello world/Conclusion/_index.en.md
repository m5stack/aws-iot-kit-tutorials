+++
title = "Conclusion"
weight = 50
pre = "<b>e. </b>"
+++

Hopefully you enjoyed the journey of building your cloud connected blinky project. Using PlatformIO, you were able to configure, build, flash, and monitor your Core2 for AWS IoT EduKit's firmware. Your device is now registered as an AWS IoT thing in your AWS account for secure cloud connectivity using on-board secure hardware. You connected to AWS IoT Core, sent messages from the device over MQTT, and received an MQTT message from the AWS IoT console's MQTT client which was used to toggle the lights on the device.

While the AWS IoT EduKit program does not teach C or require it, it is critical for embedded development. The control and flexibility from the lower level code allows you to gain the maximum potential out of your hardware. If you are not familiar with C programming but curious to learn or a bit rusty and want dive into the application code we provide in this program, Jens Gustedt provides [Modern C](https://modernc.gforge.inria.fr/) under the Creative Commons license and is free to [download](https://modernc.gforge.inria.fr/download.html). 

## What's Happening on the Device
## The FreeRTOS Kernel
The application on the device runs using the [FreeRTOS kernel](https://www.freertos.org/)—[a real-time operating system](https://www.freertos.org/about-RTOS.html) (RTOS). It is bundled into the binary you compiled (as well as other libraries and drivers) with your application code to run on. The FreeRTOS kernel schedules a block of sequential code (a [task](https://www.freertos.org/taskandcr.html)) to have time in the processor to execute. There are tasks that typically loop forever to create messages to be sent to AWS, process received messages, update the display, blink the LEDs, etc. 

The Core2 for AWS IoT EduKit packs a 240MHz dual core processor, however, it pales in comparison to the 8+ core at 4+GHz computers common today. To make the most out of the precious processor time, the FreeRTOS kernel puts [tasks in a states](https://www.freertos.org/RTOS-task-states.html) such as running or suspended. The FreeRTOS kernel gives microcontroller applications the ability to optimize the processor by rapidly switching between individual tasks to run once another task has put itself into a suspended or blocked state (see diagram below). This gives the appearance of concurrency and greatly improves an embedded application's performance when compared to a simpler [super loop application](https://en.wikibooks.org/wiki/Embedded_Systems/Super_Loop_Architecture). 
{{< img "RTOS_scheduling_execution.png" "Example of FreeRTOS application scheduling" >}}

While super loop applications are simpler to program, areas of code that block/delay execution of code means that the MCU sits idle. This makes RTOS' essential for cloud connectivity with variable latency and bandwidth, applications where not losing data or performing an action due to blocking code is critical, or simply driving a smooth display screen.
{{< img "super_loop_execution.png" "Example of super loop application execution" >}}

To learn more about RTOS' and FreeRTOS, view [this video series](https://www.youtube.com/watch?v=F321087yYy4) and visit [FreeRTOS.org](https://www.freertos.org/RTOS.html).

## AWS IoT Device SDK for Embedded C
The [AWS IoT Device SDK for Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) used in this example provided simplified connectivity and messaging to/from AWS IoT and was able to work with the included secure element library to make device authentication easy. The Device SDK for Embedded C is located in the `Core2-for-AWS-IoT-EduKit/Blink-Hello-World/components/esp-aws-iot/` folder.

## The Board Support Package — Device Drivers
The device's hardware features such as the screen, power management chip, secure element, speaker, microphone, 6-axis [inertial measurement unit](https://en.wikipedia.org/wiki/Inertial_measurement_unit) (IMU), touch driver, LED bar, and more are bundled as a board support package (BSP). The BSP are drivers with APIs that provides a simplified way to accessing the features of the hardware. The BSP is located in the `Core2-for-AWS-IoT-EduKit/Blink-Hello-World/components/core2foraws/` folder. In this example, the SK6812 driver was used to control the side LED bars and the [LVGL library](https://docs.lvgl.io/v7/en/html/) was used to show the elements on the display.

On to the next tutorial, [**Smart Thermostat**](/en/smart-thermostat.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}