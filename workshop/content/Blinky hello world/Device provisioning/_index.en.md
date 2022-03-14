+++
title = "Device Provisioning"
weight = 20
pre = "<b>b. </b>"
+++

In this section, you provision your AWS IoT EduKit to connect to AWS IoT Core using the on-board Microchip ATTECC608 Trust&GO secure element to establish a [TLS connection](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html). The built-in hardware *root of trust* provides a simplified and expedited provisioning path while not exposing the private key. You can retrieve the device certificate ([public key](https://en.wikipedia.org/wiki/Public-key_cryptography)) that is built into the device to create a AWS IoT thing (a virtual representation and record of your device). The secure element's unique serial number is used as the client ID to register and identify the device in AWS IoT Core. You can use similar processes to automate the fleet deployment of thousands or millions of devices at a time.

## Open the Blink Hello World project
Complete the following steps to open the  Blinky-Hello-World project: 
1. Open **VS Code**, if necessary.
1. Expand the **File menu** and select **New Window** in the VS Code to open a new window. This provides a clean file Explorer and working environment.
1. Select the **PlatformIO logo** in the VS Code activity bar, choose **Open**, and then select **Open Project**.
1. Navigate to the `Core2-for-AWS-IoT-EduKit/Blinky-Hello-World` folder and choose **Open "Blinky-Hello-World**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Open the PIO menu, 2 - Open the PIO home, 3 - Open the project folder" >}}

## Retrieve the device certificate and register your AWS IoT thing
To create a secure TLS connection over [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) with AWS IoT Core, you need to register a thing, attach the [device certificate](https://docs.aws.amazon.com/iot/latest/developerguide/register-device-cert.html) (public key) to the thing, and attach a [security policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) to the certificate. This process ensures that rogue devices or rogue operations are not performed within your AWS account.

## Retrieving the Device Certificate and Registering your AWS IoT thing
To create a secure TLS connection over [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) to AWS IoT Core, you need to register a thing, [attach the device certificate](https://docs.aws.amazon.com/iot/latest/developerguide/register-device-cert.html) (public key) to the thing, and attach a [security policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) to the certificate to ensure rogue devices or rogue operations are not performed within your AWS account. With the inclusion of a secure element on the Core2 for {{<awsEdukitShort-en>}} reference hardware, we can automate the device registration process without ever exposing or handling sensitive private keys. Included in the project is a script that automates the process. The script retrieves the pre-provisioned device certificate from the reference hardware's secure element, places the device certificate and additional device metadata into a manifest file and signs the manifest file with a locally generated [X.509 certificate](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html#x509-client-cert-basics). The script then uses the [Microchip TrustPlatform tools](https://github.com/MicrochipTech/cryptoauth_trustplatform_designsuite) to read the manifest file from disk, verify the contents have not been tampered by checking the signature using the X.509 certificate, performs a [just-in-time registration](https://aws.amazon.com/blogs/iot/just-in-time-registration-of-device-certificates-on-aws-iot/) of the Microchip (secure element manufacturer) Certificate Authority (CA), registers the thing in AWS IoT with the device certificate, attaches a secure policy to the AWS IoT thing, and adds the AWS IoT MQTT broker endpoint address to the device firmware configuration. This process can be scaled to onboard tens of thousands of devices to AWS IoT at a time.

The secure element on the Core2 for AWS IoT EduKit reference hardware allows you to automate the device registration process without exposing or handling sensitive, private keys. The script to automate this process is included in the project folder and performs the following tasks:
* The script retrieves the pre-provisioned device certificate from the reference hardware's secure element, places the device certificate and additional device metadata into a manifest file, and signs the manifest file with a locally generated [X.509 certificate](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html#x509-client-cert-basics). 
* The script uses the [Microchip TrustPlatform tools](https://github.com/MicrochipTech/cryptoauth_trustplatform_designsuite) to read the manifest file from disk, verify that the contents have not been altered (tampered), and registers the microchip (secure element) using a [just-in-time registration](https://aws.amazon.com/blogs/iot/just-in-time-registration-of-device-certificates-on-aws-iot/) process and the microchip manufacturer's Certificate Authoritiy (CA).
    * During the registration process, the AWS IoT creates a thing, creates a device certificate and attaches it to the thing, creates a secure policy and attaches it to the certificate, and adds the AWS IoT MQTT broker endpoint address to the device's firmware configuration. 

Complete the following steps to register your AWS IoT EduKit with AWS IoT Core:
1. Connect your AWS IoT EduKit to your host machine. Be sure that the device is powered on -- if you flashed the firmware from the Getting Started tutorial, your device will make a quiet clicking sound when it's powered on.
1. Issue the following commands in the [PlatformIO CLI terminal window](prerequisites.html#open-the-platformio-cli-terminal-window) to run the registration helper script:

```bash
cd Blinky-Hello-World
pio run -e core2foraws-device_reg -t register_thing
```

## Conclusion
In this section, you used the secure element and registration script to create an AWS IoT [thing](https://docs.aws.amazon.com/iot/latest/developerguide/thing-registry.html), create a permissions [policy](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html) for your thing, attach the policy to the device certificate, and attach the device certificate to the thing. All of this was done without exposing the private key and your device was was protected against being compromised.

Now that your AWS IoT EduKit has been provisioned in AWS IoT Core, continue to [**Connecting to AWS IoT Core**](connecting-to-aws.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}