+++
title = "Device Provisioning"
weight = 20
pre = "<b>b. </b>"
+++

In this chapter, you'll provision the device for connectivity to AWS IoT Core using the on-board Microchip ATTECC608 Trust&GO secure element to establish a [TLS connection](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html). The built-in hardware root of trust allows you to have a simplified and expedited provisioning path while never exposing the private key. You can retrieve the device certificate ([public key](https://en.wikipedia.org/wiki/Public-key_cryptography)) that is built into the device to create a AWS IoT thing (a representation and record of your device). The secure element's unique serial number will be used as the client Id to register and identify the device in AWS IoT Core. You can use similar processes to automate the fleet deployment of thousands or millions of devices at a time.

## Opening the Blink Hello World project
If you have any other project already open in VS Code, first open a new window (**File** → **New Window**) to have a clean file Explorer and working environment.

For this tutorial, you will use the Blinky-Hello-World project. In your new VS Code window, click the **PlatformIO logo** in the VS Code activity bar (left most menu), select **Open** from the left PlatformIO menu, click **Open Project**, navigate to the `Core2-for-AWS-IoT-EduKit/Blinky-Hello-World` folder, and click **open**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Open PIO menu, 2 - Open PIO home, 3 - Open the project folder" >}}

## Retrieving the Device Certificate and Registering your AWS IoT thing
To perform a secure TLS connection over [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) to AWS IoT Core, you need to register a thing, [attach the device certificate](https://docs.aws.amazon.com/iot/latest/developerguide/register-device-cert.html) (public key) to the thing, and attach a [security policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) to the certificate to ensure rogue devices or rogue operations are not performed within your AWS account. With the inclusion of a secure element on the Core2 for AWS IoT EduKit reference hardware, we can automate the entire device registration without ever exposing or handling sensitive private keys. Included in the project is a script that automates the process of retrieving the pre-provisioned device certificate from the reference hardware's secure element, generates a device manifest by signing the device certificate with an [X.509 certificate](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html#x509-client-cert-basics), performs a [just-in-time registration](https://aws.amazon.com/blogs/iot/just-in-time-registration-of-device-certificates-on-aws-iot/) of the Microchip (secure element manufacturer) CA, registers the thing in AWS IoT with the device certificate, and attaches a secure policy to the AWS IoT thing.

To run the provided script using the [PlatformIO CLI terminal window](prerequisites.html#open-the-platformio-cli-terminal-window), expand the instructions for your host machine's OS:
{{%expand "Ubuntu or macOS" %}}
To run the registration helper script (located in the **utilities\AWS_IoT_registration_helper** directory), copy and paste the command below into you the PlatformIO CLI terminal window. Before executing, replace  **<<DEVICE_PORT>>** with the port your device is virtually mounted on. To look up the port again, see the instructions for [Ubuntu](../getting-started/prerequisites/linux.html#identifying-the-device-communication-port) or [macOS](../getting-started/prerequisites/macos.html#identifying-the-device-communication-port).
```bash
cd utilities/AWS_IoT_registration_helper
python3 registration_helper.py -p <<DEVICE_PORT>>
```

With the device successfully registered and provisioned in AWS IoT, go back to the **Blinky-Hello-World** directory by entering:
```bash
cd ../..
```
{{% /expand%}}
{{%expand "Windows" %}}
To run the registration helper script (located in the **utilities\AWS_IoT_registration_helper** directory), copy and paste the command below into you the PlatformIO CLI terminal window. Before executing, replace  **<<DEVICE_PORT>>** with the communications port your device is virtually mounted on. To look up the port again, see the instructions for [Windows](../getting-started/prerequisites/windows.html#identifying-the-device-communication-port).
```PowerShell
cd utilities\AWS_IoT_registration_helper\
python registration_helper.py -p <<DEVICE_PORT>>
```

With the device successfully registered and provisioned in AWS IoT, go back to the **Blinky-Hello-World** directory by entering:
```PowerShell
cd ..\..
```
{{% /expand%}}

## Chapter Conclusion
In this chapter, you used the secure element and registration script to create an AWS IoT [thing](https://docs.aws.amazon.com/iot/latest/developerguide/thing-registry.html), set up a permissions [policy](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html) for your thing, and attached the device certificate to the thing. All of this was done without ever exposing the secret private key and removing a potential avenue for it to be compromised.

On to [**Connecting to AWS IoT Core**](connecting-to-aws.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}