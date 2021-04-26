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

## Compiling and Uploading the Smart Thermostat Firmware
A sample application has already been prepared for you to build and upload to your device using Visual Studio Code and the PlatformIO extension. If you have any other project already open in VS Code, first open a new window (File → New Window) to have a clean file Explorer and working environment.

For this tutorial, you will use the Smart-Thermostat project. In your new VS Code window, 
1. Click the **PlatformIO logo** in the VS Code activity bar (left most menu)
2. Select **Open** from the left PlatformIO menu
3. Click **Open Project**
4. Navigate to the `Core2-for-AWS-IoT-EduKit/Smart-Thermostat` folder, and click **open**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Open PIO menu, 2 - Open PIO home, 3 - Open the project folder" >}}

Next, you'll need to open a new PlatformIO CLI terminal window in VS Code by:
1) Click the **PlatformIO logo** on the VS Code activity bar (left most menu).
2) From the **Quick Access** menu, under **Miscellaneous**, select **New Terminal**.

{{< img "pio-new_terminal-smart_thermostat.en.png" "PlatformIO CLI terminal in VS Code" "1 - Open PIO menu, 2 - Open new PIO Terminal, 3 - Verify you're in the 'PlatformIO CLI' terminal session, 4 - Paste the commands into terminal, 5 - If you encounter an error autodetecting the port, open the Platform.ini file and follow instructions to manually add the serial port.">}}


 To copy the configuration from the Blinky project, compile the device firmware, and upload it to your device, follow the steps for your host machine's OS to complete this chapter:

{{%expand "Ubuntu or macOS" %}}
1. Copy your `sdkconfig` file from the **Blinky-Hello-World** demo to simplify setting your Wi-Fi and AWS endpoint:
   ```bash
   cp ../Blinky-Hello-World/sdkconfig .
   ```
   {{% notice note %}}
   You can optionally manually set your Wi-Fi credentials and AWS endpoint for your firmware by following the **[Configuring the ESP32 Firmware](/en/blinky-hello-world/connecting-to-aws.html#configuring-the-esp32-firmware)** step from the **Blinky Hello World** example.
   {{% /notice %}}
2. Use the command below to compile your firmware, upload the firmware, and monitor the serial output of your device. It will take some time to build and flash the app, but after that's done you should see the stream of device logs in your terminal. You can close the monitor session with the **Ctrl** + **C** keystroke combination:
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```
{{% /expand%}}
{{%expand "Windows" %}}
1. Copy your `sdkconfig` file from the **Blinky-Hello-World** demo to simplify setting your Wi-Fi and AWS endpoint:
   ```bash
   copy ..\Blinky-Hello-World\sdkconfig .
   ```
   {{% notice note %}}
   You can optionally manually set your Wi-Fi credentials and AWS endpoint for your firmware by following the **[Configuring the ESP32 Firmware](/en/blinky-hello-world/connecting-to-aws.html#configuring-the-esp32-firmware)** steps from the **Blinky Hello World** example.
   {{% /notice %}}
2. Use the command below to compile your firmware, upload the firmware, and monitor the serial output of your device. It will take some time to build and flash the app, but after that's done you should see the stream of device logs in your terminal. You can close the monitor session with the **Ctrl** + **C** keystroke combination:
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```
{{% /expand%}}

## Validation
Before moving on to the next chapter, you can validate that your device is configured as intended by viewing the serial output in the terminal window which look like the following: 

```
I (16128) shadow: On Device: roomOccupancy false
I (16132) shadow: On Device: hvacStatus STANDBY
I (16137) shadow: On Device: temperature 64.057533
I (16143) shadow: On Device: sound 8
```

If these are working as expected, let's move on to [**Data sync**](/en/smart-thermostat/data-sync.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}