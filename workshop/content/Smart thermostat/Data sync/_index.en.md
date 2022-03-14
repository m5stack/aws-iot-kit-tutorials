+++
title = "Data sync"
weight = 30
pre = "<b>c. </b>"
+++

## Chapter introduction
By the end of this chapter, your Core2 for {{<awsEdukitShort-en>}} reference hardware kit should do the following:
In this section, you publish the sampled ambient noise level and room temperature to AWS IoT Core every ten seconds, and your device's shadow in AWS IoT Core.

## Messaging
To accomplish these goals, connect your device to your AWS IoT Core endpoint, and exchange messages between the device and the cloud in a model called publish and subscribe (PubSub). AWS IoT Core is heavily based on a protocol called [Message Queueing Telemetry Transport](https://mqtt.org/) (MQTT). From the MQTT.org home page:

> MQTT is an OASIS standard messaging protocol for the Internet of Things (IoT). It is designed as an extremely lightweight publish/subscribe messaging transport that is ideal for connecting remote devices with a small code footprint and minimal network bandwidth. MQTT today is used in a wide variety of industries, such as automotive, manufacturing, telecommunications, oil and gas, etc. 

The messages exchanged between your device and the cloud can include text, numbers, binary, or [JSON](https://www.json.org/json-en.html). JSON is a best practice for exchanging messages between decoupled systems and a recommended pattern for exploring AWS IoT services.

Given permissions, messages publish to a topic and other clients on AWS IoT Core can subscribe and receive copies of published messages. Topics can look like the following examples:

* `this is a topic`
* `domain/stuff/thing/whatever`
* `dt/device123/temperature`
* `tokyo/weather/report`
* `$aws/things/edukit/shadow/update`

The MQTT protocol defines topic partitions as separated by the forward slash '/' character. This is a foundational concept in designing topic architecture for an IoT solution. This module will help you implement topics to follow best practices. For more information, see [Designing MQTT Topics for AWS IoT Core](https://docs.aws.amazon.com/whitepapers/latest/designing-mqtt-topics-aws-iot-core/designing-mqtt-topics-aws-iot-core.html).

In your smart thermostat solution, you use a feature of AWS IoT Core called device shadows. Simply put, a device shadow is a JSON document used to store and retrieve current state information for a device. You can store key-value pairs in a JSON document, publish the document on a special topic, and AWS IoT Core stores the document in the cloud. It's a useful way to keep the latest device state changes in the cloud so that other systems can receive updates in real time. It's also useful for other systems to send desired commands back to your device because AWS IoT Core keeps your device in sync with any new desired commands stored in the device shadow. 

For example, the smart thermostat needs to report the latest room temperature and ambient noise level to power our solution. The message your device publishes looks like the following:

```JSON
{
  "state": {
    "reported": {
      "temperature": 68,
      "noise": 10
    }
  }
}
```

When AWS IoT Core receives a message, it stores the key-value pairs until a new published message overrides them; for example, when sending a command back to your smart thermostat, another system could publish the following message to your device shadow:

```JSON
{
  "state": {
    "desired": {
      "hvacStatus": "COOLING"
    }
  }
}
```

AWS IoT Core then merges the JSON documents so that your device shadow's aggregated state appears as follows:

```JSON
{
  "state": {
    "reported": {
      "temperature": 68,
      "noise": 10
    },
    "desired": {
      "hvacStatus": "COOLING"
    }
  }
}
```

The next time that your device connects to AWS IoT Core, or if it is already connected when the command is published, it receives this aggregate state document. It is up to the device code to act on that new key-value pair `"hvacStatus": "COOLING"`. AWS IoT Core makes it easy to report new values and process desired commands.

Your smart thermostat reports the latest temperature values and noise levels. Your thermostat also receives commands and tracks the state of two more values called "hvacStatus" and "roomOccupancy". These values are determined by the cloud application in the coming sections.

## Publishing messages to AWS IoT Core
You use the AWS IoT Device SDK for Embedded C (C-SDK) to communicate between your smart thermostat device and AWS IoT Core. This is a best practice for abstracting away security, network, and data layers so you can focus on the your device and solution's application logic. The C-SDK bundles libraries to connect to AWS IoT Core over the MQTT protocol, interface with the hardware secure element to sign requests, and integrate with higher order features like the device shadow.

Let's look at a few critical lines of code and analyze what they do.

The following code defines a new variable called `temperatureHandler` of type `jsonStruct_t`. We use this variable as a tool to pack individual key-value pairs and make them available to use with the device shadow. This variable also indicates the callback function to use (if any) if a new "desired" message arrive for the indicated `pKey`. 
```C
jsonStruct_t temperatureHandler;
temperatureHandler.cb = NULL;
temperatureHandler.pKey = "temperature";
temperatureHandler.pData = &temperature;
temperatureHandler.type = SHADOW_JSON_FLOAT;
temperatureHandler.dataLength = sizeof(float);
```
This example defines a new device shadow key (`temperature`), sets an initial `pData` value to `temperature`, and indicates that it is type `SHADOW_JSON_FLOAT`. You use this later when you register shadow delta behavior and publish the "reported" values to IoT Core.



The following example registers the delta behavior with the AWS IoT Device SDK. It passes the reference to the initialized SDK client and the `jsonStruct_t`value. The `jsonStruct_t` value represents the client's key-value pair that we want to report changes. The example also registers the `roomOccupancyActuator`. This is processed when the client receives a new value for `state.desired.roomOccupancy`.
```C
rc = aws_iot_shadow_register_delta(&iotCoreClient, &roomOccupancyActuator);
```


The following example demonstrates to read from the MPU6886 thermostat component to get the local reading. Notice that the temperature is converted to Fahrenheit and contains a hardcoded calibration offset. You can modify this expression to work in Centigrade or a different offset if the default value (50) doesn't produce values in the range that you expect.
```C
MPU6886_GetTempData(&temperature);
temperature = (temperature * 1.8) + 32 - 50;
```


The following is an example of code that packs the values, that we want to publish to the cloud, into the shadow document for the AWS IoT shadow service. You can add or remove key-value pairs for `state.reported` here by modifying this variadic function. If you use this example, remember to update the third parameter to match the number of key-value pairs in the list!
```C
rc = aws_iot_shadow_add_reported(JsonDocumentBuffer,
  sizeOfJsonDocumentBuffer, 4, &temperatureHandler,
  &soundHandler, &roomOccupancyActuator, &hvacStatusActuator);
```


The following code publishes the marshalled shadow document as a payload over the network to IoT Core on the topic `$aws/things/<<CLIENT_ID>>/shadow/update`. `<<CLIENT_ID>>` resolves to the client ID (serial number) of your Core2 for AWS device as displayed on your device screen and in the serial monitor output.
```C
rc = aws_iot_shadow_update(&iotCoreClient, 
  CLIENT_ID, JsonDocumentBuffer,
  ShadowUpdateStatusCallback, NULL, 4, true);
```



The following code demonstrates a`while` loop `xTask` that will effectively run forever unless the network connection is dropped. After publishing the latest message to the device shadow service and responding to any delta callbacks from received messages, the task "sleeps" for the indicated amount of time before it continues to the next cycle of the loop. You can modify the `vTaskDelay()` expression to publish more or less frequently. You may find it helpful to use a faster interval like 1000ms (one second) while testing and reduce it to ten, thirty, or sixty seconds for a real deployment.
```C
while(NETWORK_ATTEMPTING_RECONNECT == rc || NETWORK_RECONNECTED == rc || SUCCESS == rc) {
...
  vTaskDelay(1000 / portTICK_RATE_MS);
}
```



The following code represents a callback function for the `hvacStatus` actuator. This is the code that issues when the device receives a new message includes the `state.desired.hvacStatus` key-value pair. Note, you don't have to do anything to accept the desired state change. It is applied automatically to the local `hvacStatusActuator` and its `pData` element.

The `IOTUNUSED()` function suppresses compiler warnings for unused parameters. 

The if/else block is used to evaluate the text value of the new `hvacStatus` key-value and uses that to determine the color of the LED strips.
```C
void hvac_Callback(const char *pJsonString, uint32_t JsonStringDataLen, jsonStruct_t *pContext) {
    IOT_UNUSED(pJsonString);
    IOT_UNUSED(JsonStringDataLen);

    char * status = (char *) (pContext->pData);

    if(pContext != NULL) {
        ESP_LOGI(TAG, "Delta - hvacStatus state changed to %s", status);
    }

    if(strcmp(status, HEATING) == 0) {
        ESP_LOGI(TAG, "setting side LEDs to red");
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0xFF0000);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0xFF0000);
        Core2ForAWS_Sk6812_Show();
    } else if(strcmp(status, COOLING) == 0) {
        ESP_LOGI(TAG, "setting side LEDs to blue");
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0x0000FF);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0x0000FF);
        Core2ForAWS_Sk6812_Show();
    } else {
        ESP_LOGI(TAG, "clearing side LEDs");
        Core2ForAWS_Sk6812_Clear();
        Core2ForAWS_Sk6812_Show();
    }
}
```
## Monitor the device serial output
Complete the following steps to start the serial monitor and monitor your device's output.
1. Open **VS Code**, if necessary.
1. Expand the **File menu** and select **New Window** in the VS Code to open a new window. This provides a clean file Explorer and working environment.
1. Select the **PlatformIO logo** in the VS Code activity bar, choose **Open**, and then select **Open Project**.
1. Navigate to the `Core2-for-AWS-IoT-EduKit/Smart-Thermostat` folder and choose **open**.
1. From the **Quick Access** menu, under **Miscellaneous**, select **New Terminal**.
1. Issue the following command into the PIO terminal window:
```bash
pio run --environment core2foraws --target monitor
```

## Validation
Complete the following steps to validate that your device is configured as intended:

1. Open the AWS IoT Core console test page, subscribe to the topic `$aws/things/<<CLIENT_ID>>/shadow/update/accepted` and you should see new messages arriving in time with your **vTaskDelay()**. (Replace <<CLIENT_ID>> with your device client Id/serial number printed on the screen.)
2. Open the AWS IoT Core console test page, publish a new shadow message on the topic `$aws/things/<<CLIENT_ID>>/shadow/update`. You should see the Core for {{<awsEdukitShort-en>}}'s LED bars change from blue, to red, to off to represent the **COOLING**, **HEATING**, and **STANDBY** published values. See below for a sample shadow message. Test the effects by toggling the **hvacStatus** (set to **HEATING** or **COOLING**) and/or **roomOccupied** values (set to **true** or **false**) each time you publish the message.

1. Log into your AWS account.
1. Navigate to the **[AWS IoT console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/)**
1. Select **Test** in the navigation pane to open the client view.
1. Confirm that the *Subscribe to a topic* tab is active in the MQTT test client window.
1. Replace *`<<CLIENT_ID>>`* in the following topic with your device ID, enter it into the **Topic filter** field, and choose **Subscribe**. It may take a couple minutes for new messages to arrive. When they do, you should see new messages arriving periodically (as defined by **vTaskDelay()**).
```bash
  $aws/things/<<CLIENT_ID>>/shadow/update/accepted
```
6. After you see messages arriving in the client view, choose the **Publish to a topic** tab. 
1. Replace *`<<CLIENT_ID>>`* in the following topic with your device ID, enter it into the **Topic name** field, and choose **Publish**. It may take a couple minutes for new messages to arrive. When they do, you should see new messages arriving periodically (as defined by **vTaskDelay()**).
```bash
  $aws/things/<<CLIENT_ID>>/shadow/update
```

8. Before you choose publish, replace the information in the Message payload field with the following. Then, choose Publish. You should see the AWS IoT EduKit's LED bars change from blue to red and off to represent the `COOLING`, `HEATING`, and `STANDBY`states. The following displays a sample shadow message. 
```
{ "state": { "desired": { "hvacStatus": "HEATING", "roomOccupancy": true } } }
```

9. See what happens when you change `hvacStatus` to `HEATING` or `COOLING. Also, see what happens when you change `roomOccupied` to `true` or `false`. Be sure to publish a new message each time you make a change.



Now that you can receive readings from your AWS IoT EduKit and change the HVAC states, continue to [**Data transforms and routing**](/en/smart-thermostat/data-transforms-and-routing.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}