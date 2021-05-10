+++
title = "Blinking the LEDs"
weight = 40
pre = "<b>d. </b>"
+++

At this point, your device software is running and you are connected to AWS IoT Core, are sending messages, and are ready to receive and act on messages from the cloud. In this chapter, you will subscribe to an [MQTT topic](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html), view the messages coming into AWS IoT Core, and send a message to blink your device LEDs. Since MQTT is a [publish-subscribe protocol](https://mqtt.org/), you can subscribe and/or publish to specific topics. The policy that was set earlier in the registration script constrained the topics that the device can subscribe and publish to. This is critical for filtering, and potentially for security. Your device can only send or receive to topics that matches the client Id, which in this case is the same as the unique serial number of the device's secure element.

## Subscribing via the AWS IoT MQTT client
The AWS IoT MQTT client in the AWS IoT Core console allows you to both view and publish MQTT messages. To start, go to [AWS IoT console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/) and choose **Test** to open the client view.

{{< img "aws_iot-mqtt_test_client-subscribe.en.png" "Choose test in AWS IoT console" >}}

In the **Subscription topic** field, enter `<<CLIENT_ID>>/#` to subscribe to all MQTT topic names. You can copy and paste the client Id from the serial output in PIO terminal window, or retrieve it from the device screen. The multi-level wild card [topic filter](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#topicfilters) is **#** and can only be used once as the last character of a topic filter to subscribe to all topics that matches the string before it. Once you press the **Subscribe** button, you will see messages being sent from your device. The device is only allowed to publish messages to the topic beginning with `<<CLIENT_ID>>/`. This gives the ability for another subscriber (e.g. cloud application) to filter topics for specific client devices, and can also be narrowed to be more specific topics by using **/**'s as hierarchy separators. For example, **01234EXAMPLE/sensor/temperature/f/** would provide a Fahrenheit temperature reading from a specific device with the client Id 01234EXAMPLE.

## Blinking the LED
To start/stop the blinking of the LED bars on the sides of the M5Stack Core2 for AWS IoT EduKit reference hardware, we're going to publish from the console's AWS IoT MQTT client on a topic that the Core2 for AWS IoT EduKit is subscribed to. The subscription topic for the device must start with `<<CLIENT_ID>>/`.

In the **Publish to a topic** tab, enter the command below, first replacing the **<<CLIENT_ID>>** text with your device client Id and press the **Publish** button:
```
<<CLIENT_ID>>/blink
```
{{< img "aws_iot-mqtt_test_client-publish.en.png" "Subscribing to messages and publishing with AWS IoT console MQTT client" >}}
Your device should now have the side bar LEDs blinking. To pause the blinking, simply press the **Publish** button again to the same topic.

{{% notice info %}}
To exit the serial monitor, press **CTRL** + **C**.
{{% /notice %}}

## Cleaning up
To optimize resources used and avoid unwanted possible AWS Cloud service charges, you will be erasing the flash on your device to get it ready for the subsequent tutorials. To erase the flash, you'll need to be in a project that has already been built (such as the one you just completed), exit a running serial monitor that might block the port (press **CTRL** + **C**), and use the command:

```
pio run --environment core2foraws --target erase
```


On to the final chapter, the [**conclusion**](conclusion.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}