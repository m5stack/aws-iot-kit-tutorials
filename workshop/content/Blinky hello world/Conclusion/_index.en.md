+++
title = "Conclusion"
weight = 50
pre = "<b>e. </b>"
+++

We hope that you enjoyed building your cloud connected blinky project. 

Through this tutorial, you configured, built, flashed, and monitored your Core2 for AWS IoT EduKit's firmware using PlatformIO, Your device is registered as an AWS IoT thing in your AWS account and utilizes secure cloud connectivity using on-board secure hardware. Finally, you connected to AWS IoT Core, sent messages from the device over MQTT, and received an MQTT message from the AWS IoT console's MQTT client to toggle the device's lights on and off.

While the AWS IoT EduKit program does not teach C programming, or require it as a prerequisite, it is critical to understand for embedded development. The control and flexibility provided through low-level code allows you to gain maximum potential from your hardware. If want dive into the application code we provide in this program and are curious to learn more about C programming., Jens Gustedt provides a free course called [Modern C](https://gustedt.gitlabpages.inria.fr/modern-c/) under the Creative Commons license. Its an excellent course and free to [download](https://hal.inria.fr/hal-02383654/document). 

Let's take a minute to look a little deeper into what you accomplished through this tutorial.

## What's happening on the device
### The FreeRTOS Kernel
The application on the device runs uses the [FreeRTOS kernel](https://www.freertos.org/) (for more information, see [What is An RTOS?](https://www.freertos.org/about-RTOS.html)). It bundls into the binary (application code) that you compiled with other libraries and drivers. The FreeRTOS kernel operates as a scheduler and organizes blocks of sequential code as a [task](https://www.freertos.org/taskandcr.html) to give each task a share of processor runtime. There are tasks that typically loop forever to create messages that are sent to AWS, process received messages, update the display, and blink the LEDs. 

We are proud that the AWS IoT EduKit contains a 240MHz dual core processor. However, this pales in comparison to today's computers that contain 4+GHz processors with 8+ cores. To make the most out of the precious processor resources in the AWS IoT EduKit, the FreeRTOS kernel maintains [tasks in a state](https://www.freertos.org/RTOS-task-states.html); such as, running or suspended. The FreeRTOS kernel gives microcontroller applications the ability to optimize the processor by rapidly switching between individual tasks. A task can run after another task is in a suspended or blocked state (see diagram below). 
{{< img "RTOS_scheduling_execution.en.png">}}
The following outlines the scheduling process in an RTOS kernel example in the above diagram: 
1. The *MQTT task* transitions to a running state to send or receive messages.
1. The *MQTT task* transitions to a blocked state and the *display task* transitions to running. 
1. The *display task* is blocked and the *MQTT task* starts running.
1. The *MQTT task* is blocked and the *blink task* starts running. 
1. The *blink task* is blocked and the*display task* starts running.
1. The *display task* is blocked and the *blink task*starts running.
1. the blink task is blocked and the *display task* starts running. 

The RTOS kernel gives the appearance of concurrency and greatly improves an embedded application's performance when compared to a simpler [super loop application](https://en.wikibooks.org/wiki/Embedded_Systems/Super_Loop_Architecture). 

While super loop applications are simpler to program, areas of code that block/delay execution of other code cause the MCU to sit idle. A device connecting to the AWS cloud will have variable round-trip latency and bandwidth constraints that make the performance advantage of using an RTOS necessary. An RTOS provides significant advantages for other applications where it's paramount to not lose data or to be able to perform a critical operation without being obstructed by another code block's execution (e.g. health and safety), or to simply drive a smooth display screen.
{{< img "super_loop_execution.en.png">}}
The following outlines the scheduling process in an Super Loop example in the above diagram: 
1. The *MQTT* message interrupt starts to send and receive messages.
1. The *MQTT* process finishes, and the *blink code* starts and turns on the LEDs. Code execution pauses for 500ms, the LEDs toggle off, and code execution pauses for 500ms.
1. The *blink code* finishes and the *display code* starts. 
1. The *display code* finishes and the LEDS toggle on. Code execution pauses for 500ms, the LEDs toggle off, and code execution pauses for 500ms.
1. The *blink code* finishes and the *display code* starts. 


To learn more about RTOS' and FreeRTOS, see the [Introduction to RTOS](https://www.youtube.com/watch?v=F321087yYy4) video series and visit [FreeRTOS.org](https://www.freertos.org/RTOS.html).

### AWS IoT Device SDK for Embedded C
The [AWS IoT Device SDK for Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) used in this tutorial provided simplified connectivity and messaging to and from AWS IoT. It also worked with the included secure element library to make device authentication easy. To review and learn more, the Device SDK for Embedded C is located in the `Core2-for-AWS-IoT-EduKit/Blink-Hello-World/components/esp-aws-iot/` folder and see [AWS IoT Device SDK for Embedded C](https://docs.aws.amazon.com/freertos/latest/userguide/c-sdk.html).

### The Board Support Package — Device Drivers
The device's hardware features are bundled as a board support package (BSP). The BSP are drivers with APIs that provide a simplified way to access the hardware features. Feature examples include, the screen, power management chip, secure element, speaker, microphone, 6-axis [inertial measurement unit](https://en.wikipedia.org/wiki/Inertial_measurement_unit) (IMU), touch driver, and LED bar. 

You used the BSP APIs when you controlled the side LED bars using the SK6812 driver. You also used the [LVGL library](https://docs.lvgl.io/v7/en/html/) to show the elements on the device's display. To review and learn more, the BSP is located in the `Core2-for-AWS-IoT-EduKit/Blink-Hello-World/components/core2foraws/` folder and see [Core2 for AWS IoT EduKit BSP](http://localhost:1313/en/api-reference/). 

On to [**Smart Thermostat**](/en/smart-thermostat.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}