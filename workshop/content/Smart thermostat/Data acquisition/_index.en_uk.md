+++
title = "Data acquisition"
weight = 20
pre = "<b>b. </b>"
+++

## Chapter introduction
By the end of this chapter, your device will do the following:

* Power on and start the local smart thermostat application
* Report to the logs with the current temperature sensed, ambient noise level measured, an HVAC status of heating/cooling/standby, and an indicator of room occupation

The Core2 for AWS IoT EduKit reference hardware kit has several sensors ready to use. For this solution, you will take readings from the temperature sensor and microphone using local application code. The application will keep track of sensor readings and status flags that are used to render a summary to the display.

## How to program the thermostat application
Your smart thermostat will sample the integrated sensors using implementations already created and included in the bundled software components. In this first step, you will simply capture the values and print them out to the logger before we move on to publishing sensor values up to AWS IoT Core.

Reading from the temperature sensor of the included MPU6886 module is easy. Here's a code snippet:

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

If you were to build, flash, and monitor your device logs with this code, the device would take one reading from the temperature sensor, write it to the logger, and stop. To continuously sample every second without blocking other processes, you would create a separate **[FreeRTOS task](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/system/freertos.html#_CPPv423xTaskCreatePinnedToCore14TaskFunction_tPCKcK8uint32_tPCv11UBaseType_tPC12TaskHandle_tK10BaseType_t)** and pin it to an MCU core, like so:

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

## Code sample
A sample application has already been prepared for you to build and deploy to your device. Follow these steps to complete this chapter.

1. Clone the code repo for this tutorial if you haven't already from another tutorial: 
   ```bash
   git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
   ```
2. Change directory to the **Smart-Thermostat** project: 
   ```bash
   cd Core2ForAWS-AWS-IoT-EduKit/Smart-Thermostat
   ```
3. Copy your `sdkconfig` file from the **Blinky-Hello-World** demo to simplify setting your Wi-Fi and AWS endpoint:
   {{%expand "macOS & Linux" %}}
   ```bash
   cp ../Blinky-Hello-World/sdkconfig .
   ```
   {{% /expand%}}
   {{%expand "Windows (64-bit) Command Prompt" %}}
   ```bash
   copy ..\Blinky-Hello-World\sdkconfig .
   ```
   {{% /expand%}}
   {{% notice note %}}
   You can optionally manually set your Wi-Fi credentials and AWS endpoint for your firmware by following the **[Configuring the ESP32 Firmware](/en_uk/blinky-hello-world/connecting-to-aws.html#configuring-the-esp32-firmware)** step from the **Blinky Hello World** example.
   {{% /notice %}}
  
4. Build, flash, and start the serial monitor, replacing **<<DEVICE_PORT>>** with the serial port your Core2 for AWS IoT EduKit device is connected. (To get the serial port the device is connected to, please see the [**Blinky Hello World — Identifying the serial port on host machine**](/en_uk/blinky-hello-world/device-provisioning.html#identifying-the-serial-port-on-host-machine)): 
   ```bash
   idf.py build flash monitor -p <<DEVICE_PORT>> 
   ```
   
5. It will take some time to build and flash the app, but after that's done you should see the stream of device logs in your terminal. You can close the monitor session with the **Ctrl** + **]** keystroke combination.

{{% notice note %}}
If you do not see **(edukit)** prefix at your shell prompt, ensure you activate your conda environment by running `conda activate edukit`.
If the idf.py command is not found, add the ESP-IDF to your path with the command `. $HOME/esp/esp-idf/export.sh` (macOS/Linux) or `%userprofile%\Desktop\esp-idf\export.bat` (Windows).
{{% /notice %}}

## Validation steps
Before moving on to the next chapter, you can validate that your device is configured as intended by...

1. When your Core2 for AWS IoT EduKit is powered on and running the application, you should be able to see in your terminal window logs that look like the following: 

```
I (16128) shadow: On Device: roomOccupancy false
I (16132) shadow: On Device: hvacStatus STANDBY
I (16137) shadow: On Device: temperature 64.057533
I (16143) shadow: On Device: sound 8
```

If these are working as expected, let's move on to [**Data sync**](/en_uk/smart-thermostat/data-sync.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}