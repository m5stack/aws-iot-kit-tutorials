+++
title = "Connecting to AWS IoT Core"
weight = 30
pre = "<b>c. </b>"
+++

In this chapter you'll configure, build, and flash your device firmware—which will connect to AWS IoT Core. In order to connect and communicate to AWS IoT Core, you need to configure your device with Wi-Fi credentials and the URL of your AWS endpoint. Connectivity is simplified with the pre-provisioned certificates in the Microchip ATECC608 Trust&GO, which is assigned to our AWS IoT thing and attached to a security policy from the previous chapter. You do not need to retrieve certificates from AWS IoT Core or generate your own. To configure, build, and flash our firmware on to the Espressif ESP32-D0WD you will use the ESP-IDF.

## Configuring the ESP32 Firmware
Configuration of your source code is handled through [Kconfig](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html). Kconfig is the same configuration system used by the Linux kernel and helps simplify available configuration options (symbols) into a tree structure. Before you set the configuration, you will first need to retrieve your AWS IoT endpoint.

```bash
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```
Copy your AWS IoT endpoint without the quotes. It will look something like `3duk1t3xampl3.iot.us-west-2.amazonaws.com`. We'll use this in a moment.

You can enter the configuration menu from the **Blink-Hello-World** directory of the repository:
```bash
idf.py menuconfig
```
{{< img "hello_world-menuconfig.webp" "Configuring Core2 for AWS IoT EduKit with idf.py menuconfig" >}}
Here you will set the configuration. Use the direction keys on your keyboard to go to **Component config** --> **Amazon Web Services IoT Platform** and open **AWS IoT Endpoint Hostname** to set the string. You can paste the address you copied moments ago into the box and hit _enter_ to set that symbol. Next, go back to the configuration home screen by pressing the *ESC* key twice. Then select **AWS IoT EduKit Configuration** from the menu. Set your **WiFi SSID** and **WiFi Password** with your Wi-FI credentials. Once you are finished, press the *s* button on your keyboard to save, confirm the location of the file by pressing *enter*, followed by *q* to quit.

{{% notice warning %}}
Be sure your SSID is for a 2.4GHz network. The ESP32-D0WD on the M5Stack Core2 for AWS hardware does not support 5GHz Wi-Fi bands.
{{% /notice %}}

## Building the ESP32 Firmware
Building the firmware is easy, but can take quite a while the first time. ESP-IDF uses CMAKE as it's build system and will link all the files necessary and begin compiling the code to a flashable binary in ELF format that can be flashed on to the device.
```bash
idf.py build
```

## Erasing Old Firmware
If you completed the Getting Started or Alexa demo, it is recommended to erase the flash memory of your device. The flash is split into separate partitions where the application data, OTA data, and keys are stored. The other walk-throughs use a different partition table than used for this tutorial and could cause issues if not wiped. Replace **<<DEVICE_PORT>>** with the serial port your Core2 for AWS IoT EduKit device is connected to:
```bash
idf.py erase_flash  -p <<DEVICE_PORT>>
```

## Flashing the ESP32 Firmware and monitoring the device output over the serial port
To flash the firmware you just built and view all the outputs by the device over the serial connection, you're going to daisy chain commands and enter:
```bash
idf.py flash monitor -p <<DEVICE_PORT>>
```
Once the device is flashed, it will reboot and run the application. You should see the device connect to your Wi-Fi network, establish a secure MQTT connection to AWS IoT Core, subscribe to the preset MQTT topic, and begin sending messages.

## Chapter conclusion
In this chapter, you have successfully compiled and flash your device and are actively monitoring it's serial outputs. Using the AWS IoT Device SDK for Embedded C, the reference hardware authenticated with the MQTT message broker (AWS IoT Core) and is ready to receive messages.

You are now ready to head to the final chapter in this tutorial, [**Blinking the LED**](/en/blinky-hello-world/blinking-the-leds.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}