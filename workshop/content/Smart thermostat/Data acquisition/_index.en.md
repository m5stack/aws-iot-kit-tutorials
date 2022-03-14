+++
title = "Data acquisition"
weight = 20
pre = "<b>b. </b>"
+++

In this section, you power on and start the local smart thermostat application. The application reports collected data to the logs; including, current temperature, ambient noise level, HVAC status (heating, cooling, standby), and room occupation indicator. 


* Power on and start the local smart thermostat application.
* Report to the logs the current temperature sensed, ambient noise level measured, an HVAC status of heating/cooling/standby, and an indicator of room occupation.

The {{<awsEdukitShort-en>}} reference hardware kit has several sensors ready to use. For this solution, you will take readings from the temperature sensor and microphone using local application code. The application will keep track of sensor readings and status flags that are used to render a summary to the display.

## How to program the thermostat application
Your smart thermostat will sample the integrated sensors using code already created and included in the bundled software components. In this first step, you will simply capture the values and print them out to the logger before we move on to publishing sensor values up to {{<awsIoTCore>}}.

Reading from the temperature sensor of the included MPU6886 module is easy. Here's a code snippet:
 

The AWS IoT EduKit reference hardware kit has several sensors that are ready to use. For this tutorial, you collect readings from the temperature sensor and microphone using local application code. The application keeps track of sensor readings and status flags and are used to render a summary to the display.

## How to program the thermostat application
Your smart thermostat samples the integrated sensors use code that is created and included in the bundled software components. The first step is to capture the values and print them out to the logger before publishing sensor values to AWS IoT Core.

The following sample code illustrates capturing data from the temperature sensor module (MPU6886):

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
  temperature = (temperature * 1.8) + 32 - 50;

  // write the value to the ESP32 logger
  ESP_LOGI("thermostat", "measured temperature is: %f", temperature);
}
```

If you use this code to build, flash, and monitor your device, the device would take one reading from the temperature sensor, write it to the logger, and stop. To continuously sample every second without blocking other processes, you would create a separate [FreeRTOS task](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/system/freertos.html#_CPPv423xTaskCreatePinnedToCore14TaskFunction_tPCKcK8uint32_tPCv11UBaseType_tPC12TaskHandle_tK10BaseType_t) and pin it to an MCU core. 

The following sample code illustrates *continuously* collecting temperature readings every 1,000 milliseconds (1 second):

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
    temperature = (temperature * 1.8) + 32 - 50;
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

## Compile and upload the Smart Thermostat Firmware
A sample application has already been prepared for you to build and upload to your device using Visual Studio Code (VS Code) and the PlatformIO (PIO) extension.

Complete the following steps to upload the application to your AWS IoT EduKit:
1. Open **VS Code**, if necessary.
1. Expand the **File menu** and select **New Window** in the VS Code to open a new window. This provides a clean file Explorer and working environment.
1. Select the **PlatformIO logo** in the VS Code activity bar, choose **Open**, and then select **Open Project**.
1. Navigate to the `Core2-for-AWS-IoT-EduKit/Smart-Thermostat` folder and choose **Open Smart-Thermostat**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Open the PIO menu, 2 - Open the PIO home, 3 - Open the project folder" >}}
1. Select the **PlatformIO logo** in the VS Code activity bar, expand **Miscellaneous**, and then choose **New Terminal**.

Next, you will utilize local Wi-Fi and AWS endpoint configuration that you established during the [CloudConnected Blinky](en/blinky-hello-world.html) tutorial. 

   {{% notice note %}}
   If you did not complete the [CloudConnected Blinky](en/blinky-hello-world.html) tutorial or you prefer to set your Wi-Fi credentials and AWS endpoint manually. Follow the [Configure the ESP32 firmware](/en/blinky-hello-world/connecting-to-aws.html/#configure-the-esp32-firmware) steps from the **Blinky Hello World** tutorial.
   {{% /notice %}}

Complete the following steps to compile copy the configuration from the Blinky project, compile the device firmware, and upload it to your device. (**Expand and complete the directions that correspond to your host machine's operating system**): 

{{%expand "macOS and Ubuntu" %}}
1. Issue the following command to copy the `sdkconfig` file from the **Blinky-Hello-World** demo to use the Wi-Fi and AWS endpoint settings that you used in the Blinky tutorial:
   ```bash
   cp ../Blinky-Hello-World/sdkconfig .
   ```
2. Issue the following command to compile and upload the firmware, and monitor the serial output from your device. It will take some time to build and flash the app, but after that's complete you should see the stream of device logs in your terminal:
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```
1. When you're finished, close the monitor session by issuing the **Ctrl+C** keyboard command.
{{% /expand%}}


{{%expand "Windows" %}}
1. Issue the following command to copy the `sdkconfig` file from the **Blinky-Hello-World** demo to use the Wi-Fi and AWS endpoint settings that you used in the Blinky tutorial:
   ```bash
   copy ..\Blinky-Hello-World\sdkconfig .
   ```
2. Issue the following command to compile and upload the firmware, and monitor the serial output from your device. It will take some time to build and flash the app, but after that's complete you should see the stream of device logs in your terminal:
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```
1. When you're finished, close the monitor session by issuing the **Ctrl+C** keyboard command.
{{% /expand%}}

## Validation
Validate that your device is configured as intended by reviewing the serial output in the terminal window. Your output should look similar to the following: 

```
I (16128) shadow: On Device: roomOccupancy false
I (16132) shadow: On Device: hvacStatus STANDBY
I (16137) shadow: On Device: temperature 64.057533
I (16143) shadow: On Device: sound 8
```

Now that the serial output is configured, continue to [**Data sync**](/en/smart-thermostat/data-sync.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}