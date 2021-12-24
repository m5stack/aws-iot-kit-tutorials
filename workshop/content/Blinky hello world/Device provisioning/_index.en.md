+++
title = "Device Provisioning"
weight = 20
pre = "<b>b. </b>"
+++

In this chapter, you'll provision your device to connect to AWS IoT Core using the on-board Microchip ATTECC608 Trust&GO secure element to establish a [TLS connection](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html). The built-in hardware root of trust allows you to have a simplified and expedited provisioning path while never exposing the private key. You can retrieve the device certificate ([public key](https://en.wikipedia.org/wiki/Public-key_cryptography)) that is built into the device to create a AWS IoT thing (a representation and record of your device). The secure element's unique serial number will be used as the client Id to register and identify the device in AWS IoT Core. You can use similar processes to automate the fleet deployment of thousands or millions of devices at a time.

## Opening the Blink Hello World project
If you have any other project already open in VS Code, first open a new window (**File** → **New Window**) to have a clean file Explorer and working environment.

For this tutorial, you will use the Blinky-Hello-World project. In your new VS Code window, click the **PlatformIO logo** in the VS Code activity bar (left most menu), select **Open** from the left PlatformIO menu, click **Open Project**, navigate to the `Core2-for-AWS-IoT-EduKit/Blinky-Hello-World` folder, and click **open**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Open PIO menu, 2 - Open PIO home, 3 - Open the project folder" >}}

## Retrieving the Device Certificate and Registering your AWS IoT thing
To create a secure TLS connection over [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) to AWS IoT Core, you need to register a thing, [attach the device certificate](https://docs.aws.amazon.com/iot/latest/developerguide/register-device-cert.html) (public key) to the thing, and attach a [security policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) to the certificate to ensure rogue devices or rogue operations are not performed within your AWS account. With the inclusion of a secure element on the Core2 for AWS IoT EduKit reference hardware, we can automate the device registration process without ever exposing or handling sensitive private keys. Included in the project is a script that automates the process. The script retrieves the pre-provisioned device certificate from the reference hardware's secure element, places the device certificate and additional device metadata into a manifest file and signs the manifest file with a locally generated [X.509 certificate](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html#x509-client-cert-basics). The script then uses the [Microchip TrustPlatform tools](https://github.com/MicrochipTech/cryptoauth_trustplatform_designsuite) to read the manifest file from disk, verify the contents have not been tampered by checking the signature using the X.509 certificate, performs a [just-in-time registration](https://aws.amazon.com/blogs/iot/just-in-time-registration-of-device-certificates-on-aws-iot/) of the Microchip (secure element manufacturer) Certificate Authority (CA), registers the thing in AWS IoT with the device certificate, attaches a secure policy to the AWS IoT thing, and adds the AWS IoT MQTT broker endpoint address to the device firmware configuration. This process can be scaled to onboard tens of thousands of devices to AWS IoT at a time.

To run the registration helper script using the [PlatformIO CLI terminal window](prerequisites.html#open-the-platformio-cli-terminal-window):

```bash
cd Blinky-Hello-World
pio run -e core2foraws-device_reg -t register_thing
```

## Chapter Conclusion
In this chapter, you used the secure element and registration script to create an AWS IoT [thing](https://docs.aws.amazon.com/iot/latest/developerguide/thing-registry.html), set up a permissions [policy](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html) for your thing, and attached the device certificate to the thing. All of this was done without ever exposing the secret private key and in this way we removed a potential avenue for it to be compromised.

On to [**Connecting to AWS IoT Core**](connecting-to-aws.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}