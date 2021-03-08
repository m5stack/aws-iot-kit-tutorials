+++
title = "Flashing Espressif Certificates"
weight = 20
pre = "<b>b. </b>"
+++

## Chapter introduction
By the end of this chapter, you will have the companion phone applications installed on your mobile device, cloned the necessary code repository, received certificates for connecting to AWS IoT using Espressif Alexa's AWS account, and flashed the certificates to separate flash partitions of the reference hardware.

## Companion Mobile Applications
In order to complete the authentication with Alexa, you will need Espressif's companion app to provision the device to WiFi and provision the reference hardware with your Alexa account.

Download the ESP Alexa Phone App:
[iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

It is recommended (_optional_ for this tutorial) to have the Amazon Alexa app available for [iOS](https://apps.apple.com/us/app/amazon-alexa/id944011620) and [Android](https://play.google.com/store/apps/details?id=com.amazon.dee.app) - this is the same app used to provision most Alexa-enabled devices.

## Accessing the code
All the code for this tutorial is located in the `Alexa_for_IoT-Intro` folder from the repo you cloned in the [**Blinky Hello World**](/en_uk/blinky-hello-world.html) tutorial. To clone the repo again:
```
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```

## Set up AWS IoT Certificates 
You need to create the AWS IoT certificates to communicate with AWS IoT core. For this workshop and device, Espressif has provided the AWS IoT certificates that can be used with the M5Stack Core2 for AWS IoT EduKit reference hardware. Follow the steps [here](https://espressif.github.io/esp-va-sdk/#/) to obtain your certificates.

After unzipping the downloaded certificates, you will have a folder called **espcredentials**. Please navigate to the folder and modify **mfg_config.csv** (provided with the certificates) to reflect the correct paths for each file. Then run the commands below in the same folder to flash the certificates to device. Replace **<<DEVICE_PORT>>** with the serial port your Core2 for AWS IoT EduKit device is connected to:
```bash
python $IDF_PATH/components/nvs_flash/nvs_partition_generator/nvs_partition_gen.py generate /path/to/mfg_config.csv mfg.bin 0x6000

python $IDF_PATH/components/esptool_py/esptool/esptool.py --chip esp32 --port <<DEVICE_PORT>> write_flash 0x10000 mfg.bin
```

With everthing set up and ready, let's move on to [**Building and Testing AFI**](/en_uk/intro-to-alexa-for-iot/building-and-testing-afi.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}