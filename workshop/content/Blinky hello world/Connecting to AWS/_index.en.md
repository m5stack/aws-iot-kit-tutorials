+++
title = "Connecting to AWS IoT Core"
weight = 30
pre = "<b>c. </b>"
+++

In this chapter you'll configure, build, and flash your device firmware—which will allow your device to connect to your Wi-Fi network and to AWS IoT Core. In order to connect and communicate to AWS IoT Core, you need to configure your device with Wi-Fi credentials and the URL of your [AWS IoT endpoint](https://docs.aws.amazon.com/iot/latest/developerguide/connect-to-iot.html#iot-device-endpoint-intro). Establishing a secure MQTT connection is simplified with the [AWS IoT Device SDK for Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) and the pre-provisioned secure hardware certificates in the on-board [Microchip ATECC608 Trust&GO](https://www.microchip.com/wwwproducts/en/ATECC608B-TNGTLS). With the secure element, you do not need to retrieve certificates from AWS IoT Core, or generate your own, to connect. The connectivity libraries in the AWS IoT Device SDK for Embedded C simplify connectivity and access to AWS IoT services and features.

## Configuring the ESP32 Firmware
Configuration of your source code is handled through [Kconfig](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html). Kconfig is the same configuration system used by the Linux kernel and helps simplify available configuration options (symbols) into a tree structure. 

Now, you'll go into the KConfig menu to configure the required [symbols](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html) for the device to connect to your 2.4GHz Wi-Fi network. Start by switching to the **Blink-Hello-World** directory of the repository in the PIO terminal window and enter:
```bash
pio run --environment core2foraws --target menuconfig
```

{{< img "idf_menuconfig-wifi.en.webp" "Configuring Core2 for AWS IoT EduKit with p.py menuconfig" >}}

Use the direction keys (or *K* and *J*, or *-* and *+*) on your keyboard to go to **AWS IoT EduKit Configuration** from the menu. Set your **WiFi SSID** and **WiFi Password** with your Wi-FI credentials. Once you are finished, press the *s* button on your keyboard to save, confirm the location of the file by pressing *enter*, followed by *q* to quit.

{{% notice warning %}}
Be sure your SSID is for a 2.4GHz network. The ESP32-D0WD on the M5Stack Core2 for AWS hardware does not support 5GHz Wi-Fi bands.
{{% /notice %}}

## Building, uploading, and monitoring the Blinky Hello World firmware
You are now ready to build (compile) and upload the Blinky Hello World firmware. The process is the same as with the Getting Started tutorial for building, flashing, and monitoring the serial output:
   
1) To build the firmware, paste in the command below (it will take several minutes):
    ```bash
    pio run --environment core2foraws
    ```
2) With the build successful, it's time to upload the compiled firmware to the connected device over USB and monitor the messages sent over serial output to the host machine by running the command:
    ```bash
    pio run --environment core2foraws --target upload --target monitor
    ```

{{% notice info %}}
If during upload or while monitoring the serial output you receive an error about an incorrect port or timeout, open the `platformio.ini` file, and follow the instructions there to manually set the upload port.
{{% /notice %}}

## Chapter conclusion
In this chapter, you have successfully compiled and flashed your device and are actively monitoring it's serial outputs. Using the AWS IoT Device SDK for Embedded C, the reference hardware authenticated with the [MQTT message broker](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) (AWS IoT Core) and is ready to receive messages.

You are now ready to head to the final chapter in this tutorial, [**Blinking the LED**](/en/blinky-hello-world/blinking-the-leds.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}
