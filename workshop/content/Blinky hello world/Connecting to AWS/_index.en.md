+++
title = "Connect to AWS IoT Core"
weight = 30
pre = "<b>c. </b>"
+++

In this lesson, you will configure, build, and flash your device's firmware so that your device can communicate with AWS IoT Core. To do this, you must configure your device with your local Wi-Fi credentials and your [AWS IoT endpoint's](https://docs.aws.amazon.com/iot/latest/developerguide/connect-to-iot.html#iot-device-endpoint-intro) URL. 

Establishing a secure MQTT connection is simplified with the [AWS IoT Device SDK for Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) and the [Microchip ATECC608 Trust&GO](https://www.microchip.com/wwwproducts/en/ATECC608B-TNGTLS) pre-provisioned, secure hardware certificates. With the secure element, you do not need to retrieve certificates from AWS IoT Core or generate your own to connect. The connectivity libraries in the AWS IoT Device SDK for Embedded C simplify connectivity and access to AWS IoT services and features.

## Configure the ESP32 firmware
Your source code configuration is handled through [Kconfig](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html). The Linux kernel also uses the Kconfig configuration system to simplify their configuration options ([symbols](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html)) into a tree structure. 

Complete the following steps to open the KConfig menu:
1. Return to **VS Client** and open a new **PIO terminal** window. 
1. Navigate to the **Blinky-Hello-World** directory in the PIO terminal window. 
1. Issue **the following command** in the PIO terminal window to configure your device: 
```bash
pio run --environment core2foraws --target menuconfig
```
Complete the following steps to configure your local Wi-Fi when the Espressif IoT Development Framework Configuration window appears:

{{% notice info %}}
Navigation in the Espressif configuration window may vary based on your computer. To navigate up and down the menu, use *k* and *j*, or *-* and *+*, or *up arrow* and *down arrow*.
{{% /notice %}}


1. Navigate to **AWS IoT Kit Configuration** and press **Enter** on your keyboard. 
1. Choose **WiFi SSID**. 
1. Remove the **default SSID** (AWSWorkshop) and enter the **service set identifier (SSID) for your local Wi-Fi network** and press **Enter**.
1. Navigate to and choose **WiFi Password**.
1. Remove the **default password** and enter the **password for your local Wi-Fi network** and press **Enter**.
1. Enter **s** to save these settings, press **Enter** to confirm the file location, and press **Enter** again to close the message box.
1. Enter **q** to quit configuration and return to the PIO terminal window.

{{< img "idf_menuconfig-wifi.en.webp" "Configuring {{< awsService type="edukit-short-en" >}} with p.py menuconfig" >}}

{{% notice warning %}}
Be sure your SSID is for a 2.4GHz network. The ESP32-D0WD on the M5Stack Core2 for AWS hardware does not support 5GHz Wi-Fi bands.
{{% /notice %}}

## Build, upload, and monitor the Blinky Hello World firmware
You are now ready to build (compile) and upload the Blinky Hello World firmware. This is the same process that you performed during the [Getting Started](en/getting-started/run-rainmaker.html) tutorial. 
   
1. Issue the following command to build the firmware (**note:** this may take several minutes):
    ```bash
    pio run --environment core2foraws
    ```
2. When the build completes, issue the following command to upload the compiled firmware to the connected device through USB and monitor the messages sent over serial output to the host machine:
    ```bash
    pio run --environment core2foraws --target upload --target monitor
    ```

{{% notice info %}}
If, during upload or while monitoring the serial output, you receive an error about an incorrect port or timeout, open the `platformio.ini` file, and follow the instructions there to manually set the upload port.
{{% /notice %}}

## Conclusion
In this lesson, you successfully compiled the firmware updates, flashed your device, and monitored its serial outputs. Your {{< awsService type="edukit-short-en" >}} then used the AWS IoT Device SDK for Embedded C to authenticate with the [MQTT message broker](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) (AWS IoT Core. Your device is ready to receive messages.

You are now ready to control the LEDs on the side of your device. Continue to [**Blinking the LEDs**](/en/blinky-hello-world/blinking-the-leds.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}
