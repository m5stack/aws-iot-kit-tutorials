+++
title = "Blinking the LEDs"
weight = 40
pre = "<b>d. </b>"
+++

Through the previous sections, you configured your device software, it's running, and you are connected to AWS IoT Core. Your AWS IoT EduKit is sending messages to the cloud, and is ready to receive and act on messages from the cloud. In this section, you subscribe to an [MQTT topic](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html), view the messages that AWS IoT Core receives, and send an MQTT request to blink your device LEDs. 

MQTT is a [publish-subscribe protocol](https://mqtt.org/). This means that you can subscribe and publish to specific topics. The policy that the registration script created defines these topics. Your device can only send or receive messages to AWS IoT Core through the topics that satisfy the AWS IoT policy’s parameters and match the client ID ; in our case, the client ID is the same as the device’s secure element’s unique serial number. Understanding this process is critical so that you can filter messages and secure communication. 

## Subscribe through the AWS IoT MQTT client
The AWS IoT MQTT client in the AWS IoT Core console allows you to both view and publish MQTT messages. 

Complete the following steps to subscribe to an MQTT message topic:
1. Log into your AWS Account. 
1. Navigate to the [AWS IoT console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/) 
1. Expand **Test** in the navigation pane and choose **MQTT test client** to open the client view.

{{< img "aws_iot-mqtt_test_client-subscribe.en.png" "Choose test in AWS IoT console" >}}

4. Confirm that the *Subscribe to a topic* tab is active in the MQTT test client window.
1. Enter **#** (hash symbol) in theIn the Topic filter field to subscribe to *all* MQTT topic names. (The hash symbol, *#*, is a multi-level wild card [topic filter](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#topicfilters) and can only be used once and as the last character of a topic filter.)
1. Choose **Subscribe**. 

The messages that your device sends to the cloud will begin to appear. (**Note:** this can take a minute or two before they appear) 

Because of the policy constraints, the device can only publish messages to the topic beginning with its client ID. This provides flexibility so that another subscriber (such as, a cloud application) to filter topics for specific devices. Filters can also be applied to display more specific topics; such as, a specific device's temperature readings. 

## Blink the LED
To start or stop blinking the LED bars on the side of the AWS IoT EduKit, publish a message to the device from the AWS IoT MQTT client in a topic that the device subscribes. The subscription topic for the device will have the pattern `<<CLIENT_ID>>/#`. You can view the subscription topic on the device after it successfully subscribes to the topic. Additionally, you can view the client ID that's been output on the host machine's serial monitor.

Complete the following steps to begin blinking the LED bars on the device:
1. Begin in the AWS IoT console.
1. Expand **Manage** in the navigation pane, and choose **Things**.
1. The Things page displays all the devices that are registered in your AWS account for this Region. You should have one listed with a *microchip-tng* Thing type--this is your AWS IoT EduKit device. Write down this **device's name** because it is your AWS IoT device ID. 
1. Expand **Test** in the navigation pane, and choose **MQTT test client window**. 
1. Choose the **Publish to a topic** tab.
1. Replace **<<CLIENT_ID>>** in the following code and enter the topic into the **PubTopic name** field:
```
<<CLIENT_ID>>/blink
```
4. Leave the default message in the **Message payload** field and press **Publish**. (**Note:** the message that you sent to this topic doesn't matter. The device will turn on or off the lights based on any published message to this topic)

{{< img "aws_iot-mqtt_test_client-publish.en.png" "Subscribing to messages and publishing with AWS IoT console MQTT client" >}}


Your device's LED bars should now blink. To pause the lights, press the **Publish** button again to publish another message to the same topic.

{{% notice info %}}
To exit the PIO serial monitor, return to VS Client and press **CTRL+C**.
{{% /notice %}}

## Clean up
To avoid unwanted possible AWS Cloud service charges and optimize your resources, erase the flash memory on your AWS IoT EduKit. This will also prepare it subsequent tutorials. 

Complete the following steps to erase the flash memory on your device: 
1. Open **VS Client**, if necessary. 
1. Open the PIO terminal.
1. If a project is running, press **CTRL+C** to stop it. If there isn't a project running, continue to the next step.
1. Issue the following command in the PIO terminal: 

```
pio run --environment core2foraws --target erase
```
>  When the command in the  terminal window completes, the AWS IoT EduKit may not automatically restart or appear to be different. Power the device off and then on to confirm that the firmware was flashed.

{{% notice note %}}
Erasing the firmware results in a blank device screen and an audible ticking sound. This is expected behavior. The device is continually rebooting itself because it doesn't have an application to run.
{{% /notice %}}



Congratulations! You successfully communicated to your AWS IoT EduKit and blinked its LEDs.  Now you are ready to the complete the final section of this tutorial, [**conclusion**](conclusion.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}