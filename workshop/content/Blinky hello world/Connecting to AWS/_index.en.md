+++
title = "Connecting to AWS IoT Core"
weight = 30
pre = "<b>c. </b>"
+++

In this chapter you'll configure, build, and flash your device firmware—which will allow your device to connect your Wi-Fi network and to AWS IoT Core. In order to connect and communicate to AWS IoT Core, you need to configure your device with Wi-Fi credentials and the URL of your [AWS IoT endpoint](https://docs.aws.amazon.com/iot/latest/developerguide/connect-to-iot.html#iot-device-endpoint-intro). Establishing a secure MQTT connection is simplified with the [AWS IoT Device SDK for Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) and the pre-provisioned secure hardware certificates in the on-board [Microchip ATECC608 Trust&GO](https://www.microchip.com/wwwproducts/en/ATECC608B-TNGTLS). With the secure element, you do not need to retrieve certificates from AWS IoT Core or generate your own to connect. The connectivity libraries in the AWS IoT Device SDK for Embedded C simplifies connectivity and access to AWS IoT services and features.

## Configuring the ESP32 Firmware
Configuration of your source code is handled through [Kconfig](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html). Kconfig is the same configuration system used by the Linux kernel and helps simplify available configuration options (symbols) into a tree structure. Before you set the configuration, you will first need to retrieve your AWS IoT endpoint.

```bash
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```
Copy your AWS IoT endpoint without the quotes. It will look something like `3duk1t3xampl3.iot.us-west-2.amazonaws.com`. We'll use this in a moment.

You can enter the configuration menu from the **Blink-Hello-World** directory of the repository:
```bash
pio run --environment core2foraws --target menuconfig
```
{{< img "idf_menuconfig-aws_endpoint.en.webp" "Configuring Core2 for AWS IoT EduKit with p.py menuconfig" >}}
Here you will set the configuration. Use the direction keys on your keyboard to go to **Component config** --> **Amazon Web Services IoT Platform** and open **AWS IoT Endpoint Hostname** to set the string. You can paste the address you copied moments ago into the box and hit _enter_ to set that symbol. Next, go back to the configuration home screen by pressing the *ESC* key twice. Then select **AWS IoT EduKit Configuration** from the menu. Set your **WiFi SSID** and **WiFi Password** with your Wi-FI credentials. Once you are finished, press the *s* button on your keyboard to save, confirm the location of the file by pressing *enter*, followed by *q* to quit.

{{% notice warning %}}
Be sure your SSID is for a 2.4GHz network. The ESP32-D0WD on the M5Stack Core2 for AWS hardware does not support 5GHz Wi-Fi bands.
{{% /notice %}}

## Building, uploading, and monitoring the Blinky Hello World firmware
You are now ready to build (compile) and upload the Blinky Hello World firmware. The process is the same as with the Getting Started tutorial for building, flashing, and monitoring the serial output:
   
1) To build the firmware, paste in the command below (this will take several minutes):
    ```bash
    pio run --environment core2foraws
    ```
2) With the build successful, it's time to upload the compiled firmware to the connected device over USB by running the command:
    ```bash
    pio run --environment core2foraws --target upload
    ```
3) Lastly, monitor the serial output from the device on your host machine via:
    ```bash
    pio run --environment core2foraws --target monitor
    ```
{{% notice info %}}
If during upload or monitoring the serial output you receive an error about an incorrect ports or timeout, open the `platformio.ini` file, and follow the instructions in that file for how to manually set the upload port.
{{% /notice %}}

## Chapter conclusion
In this chapter, you have successfully compiled and flashed your device and are actively monitoring it's serial outputs. Using the AWS IoT Device SDK for Embedded C, the reference hardware authenticated with the [MQTT message broker](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) (AWS IoT Core) and is ready to receive messages.

You are now ready to head to the final chapter in this tutorial, [**Blinking the LED**](/en/blinky-hello-world/blinking-the-leds.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}