+++
title = "Device Provisioning"
weight = 20
pre = "<b>b. </b>"
+++

In this chapter, you'll provision the device for connectivity to AWS IoT Core using the on-board Microchip ATTECC608 Trust&GO secure element to establish a TLS connection. The built-in hardware root of trust allows you to have a simplified and expedited provisioning path while never exposing the private key. You can retrieve the device certificate that is built into the device and create a manifest file to create a AWS IoT thing (a representation and record of your device). This device's client Id will be registered and identified in AWS IoT Core by the secure element serial number. You can use similar processes to automate the fleet deployment of thousands or millions of devices at a time.

## Identifying the serial port on host machine
Please reference [Espressif's offical doc for establishing serial connections with the ESP32](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/establish-serial-connection.html). The port of your device will vary based on your OS. For macOS, the device is typically on `/dev/cu.SLAB_USBtoUART`. For Linux, the device is typically on `/dev/ttyUSB0` (user needs to be added to dialout group). For Windows, it will start with a `COM` and end with a number.

## Retrieving the Device Certificate and Registering your AWS IoT thing
We have simplified the process of retrieving the device certificate from the Core2 for AWS IoT EduKit reference hardware's secure element, generating a device manifest by signing the device certificate with a x.509 certificate (includes your AWS IoT registration code), registering the device in AWS IoT with the device certificate, and attaching a secure policy to the AWS IoT thing.

Go into the project's AWS IoT registration helper directory and install necessary dependencies with pip:
```bash
cd Core2-for-AWS-IoT-EduKit/Blinky-Hello-World/utilities/AWS_IoT_registration_helper/
pip3 install -r requirements.txt
```

Next, you'll need to run the Python script that executes all the steps for registering the device to your AWS account. Be sure to first replace **<<DEVICE_PORT>>** with the serial port your Core2 for AWS IoT EduKit device is connected to:
```bash
python registration_helper.py -p <<DEVICE_PORT>>
```

{{% notice info %}}
If you close your shell or open a new shell, you'll need to re-enter `conda activate edukit` to reactivate the virtual environment and run ESP-IDF's `export.sh` (macOS/Linux) or `export.bat` (Windows) to re-add the ESP-IDF tools to your path each time.
{{% /notice %}}

With the device successfully registered and provisioned in AWS IoT, go back to the **Blinky-Hello-World** directory by entering:
```bash
cd ../..
```

## Chapter Conclusion
In this chapter, you used the secure element to create an AWS IoT [thing](https://docs.aws.amazon.com/iot/latest/developerguide/thing-registry.html), set up a permissions [policy](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html) for your thing, and attached the device certificate to it. All of this was done without ever exposing the secret private key and removing a potential avenue for it to be compromised.

On to [**Connecting to AWS IoT Core**](connecting-to-aws.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}