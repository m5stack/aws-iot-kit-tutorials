+++
title = "Data sync"
weight = 30
pre = "<b>c. </b>"
+++

## Chapter introduction
By the end of this chapter, your Core2 for AWS IoT EduKit reference hardware kit should do the following:

* Publish the sampled ambient noise level and room temperature to AWS IoT Core every 10 seconds
* Subscribe to your device's shadow in AWS IoT Core to receive new commands

## Messaging concepts
In this chapter, you will connect your device to your custom AWS IoT Core endpoint, then exchange messages between the device and cloud in a model called publish and subscribe, or "PubSub." AWS IoT Core is heavily based on a protocol called [Message Queueing Telemetry Transport](https://mqtt.org/) (MQTT). From the home page:

> MQTT is an OASIS standard messaging protocol for the Internet of Things (IoT). It is designed as an extremely lightweight publish/subscribe messaging transport that is ideal for connecting remote devices with a small code footprint and minimal network bandwidth. MQTT today is used in a wide variety of industries, such as automotive, manufacturing, telecommunications, oil and gas, etc. 

The messages that are exchanged between your device and the cloud can be anything from text, to numbers, to binary, to JSON. JSON is a best practice for exchanging messages between decoupled systems and a recommended pattern for exploring AWS IoT services. 

Messages are published on a topic, which is a textual string that other clients on AWS IoT Core can subscribe to in order to receive copies of published messages. Topics can look like the following examples:

* `this is a topic`
* `foo/bar`
* `domain/stuff/thing/whatever`
* `dt/device123/temperature`
* `tokyo/weather/report`
* `$aws/things/edukit/shadow/update`

As you can see, a topic can look like any text. The MQTT protocol defines topic partitions as separated by the forward slash '/' character. This is a foundational concept in designing topic architecture for an IoT solution. This module will help you implement topics to follow best practices.

In your smart thermostat solution, you will use a feature of AWS IoT Core called device shadows. Simply put, a device shadow is a JSON document used to store and retrieve current state information for a device. You can store key-value pairs in a JSON document, publish the document on a special topic, and AWS IoT Core will store the document in the cloud. It's a useful way for keeping the latest state changes of a device in the cloud so that other systems can get updates in real time. It's also useful for other systems to send desired commands back to your device because AWS IoT Core will keep your device in sync with any new desired commands stored in the device shadow. 

For example, the smart thermostat needs to report the latest room temperature and ambient noise level to power our solution. The message published by your device will look like this:

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

When AWS IoT Core receives a message like this, it will store those key-value pairs until they are overridden by a new published message. 

To illustrate sending a command back to your smart thermostat, another system could publish a message like this one to your device shadow:

```JSON
{
  "state": {
    "desired": {
      "hvacStatus": "COOLING"
    }
  }
}
```

AWS IoT Core will merge the JSON documents such that the aggregated state of your device shadow will look like this:

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

This means when your device next connects to AWS IoT Core, or if it is already connected at the time the desired command was published, it would receive this aggregate state document. It is up to the device code to act on that new key-value pair `"hvacStatus": "COOLING"` but AWS IoT Core makes it easy to report new values and process desired commands.

Your smart thermostat will report the latest values of temperature and noise level as reviewed in the previous chapter. Your thermostat will also receive commands and track the state of two more values called "hvacStatus" and "roomOccupancy". These values will be determined by the cloud application in the coming chapters.

## How to program publishing messages to AWS IoT Core
You will use the AWS IoT Device SDK for Embedded C ("C-SDK") to communicate between your smart thermostat device and AWS IoT Core. This is a best practice for abstracting away security, network, and data layers so you can focus on the application logic of your device and solution. The C-SDK bundles libraries for connecting to AWS IoT Core over the MQTT protocol, interfacing with the hardware secure element to sign requests, and for integrating with higher order features like the device shadow.

Let's look at a few critical lines of code and analyze what they do.

```C
jsonStruct_t temperatureHandler;
temperatureHandler.cb = NULL;
temperatureHandler.pKey = "temperature";
temperatureHandler.pData = &temperature;
temperatureHandler.type = SHADOW_JSON_FLOAT;
temperatureHandler.dataLength = sizeof(float);
```

The code above defines a new variable called temperatureHandler of type `jsonStruct_t` that we use as a tool for packing individual key-value pairs. This makes the key-value pairs available for use in IoT Core device shadow and indicating the callback function to use (if any) should a new "desired" message arrive for the indicated **pKey**. This example defines a new device shadow key **temperature**, setting an initial **pData** to the value of the `temperature` variable, and indicating it is of type `SHADOW_JSON_FLOAT`. These structs are used later when registering shadow delta behavior and publishing the "reported" values up to IoT Core.

```C
rc = aws_iot_shadow_register_delta(&iotCoreClient, &roomOccupancyActuator);
```

The example above registers the delta behavior with the AWS IoT Device SDK. It passes the reference to the initialized SDK client and the `jsonStruct_t` that represents the key-value pair to which we want to respond to changes. The example registers the **roomOccupancyActuator** so when the client gets a new shadow update from the cloud that includes a new value for `state.desired.roomOccupancy`, the callback function defined in the actuator is processed.

```C
MPU6886_GetTempData(&temperature);
temperature = (temperature * 1.8)  + 32 - 50;
```

Above is an example of how you read from the MPU6886 component to get the local temperature reading. Note the conversion to Fahrenheit and hardcoded callibration offset. You can modify this expression to work in Centigrade or a different offset if this default of 50 doesn't produce values in the range of what you expect.

```C
rc = aws_iot_shadow_add_reported(JsonDocumentBuffer,          
  sizeOfJsonDocumentBuffer, 4, &temperatureHandler,
  &soundHandler, &roomOccupancyActuator, &hvacStatusActuator);
```

This code packs any values we want to publish to the cloud in to the shadow document that is expected by the IoT Core shadow service. You can add or remove key-value pairs for `state.reported` here by modifying this variadic function. Don't forget to update the third parameter to match the number of key-value pairs in the list!

```C
rc = aws_iot_shadow_update(&iotCoreClient, 
  CLIENT_ID, JsonDocumentBuffer,
  ShadowUpdateStatusCallback, NULL, 4, true);
```

This code actually publishes the marshalled shadow document as a payload over the network to IoT Core on the topic `$aws/things/<<CLIENT_ID>>/shadow/update`. The definition of **<<CLIENT_ID>>** resolves to the client Id/serial number of your Core2 for AWS device as displayed on your device screen and in the serial monitor output.

```C
while(NETWORK_ATTEMPTING_RECONNECT == rc || NETWORK_RECONNECTED == rc || SUCCESS == rc) {
...
  vTaskDelay(1000 / portTICK_RATE_MS);
}
```

The `while` loop of this **xTask** will effectively run forever unless the network connection is dropped. After publishing the latest message to the device shadow service and responding to any delta callbacks from received messages, the task will sleep for the indicated amount of time before continuing to the next cycle of the loop. You can modify the `vTaskDelay()` expression to publish more or less frequently. You may find it helpful to use a faster interval like 1000 (one second) while testing and reduce it to ten, thirty, or sixty seconds for a real deployment.

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

This code represents a callback function for our **hvacStatus** actuator. This is the code that is executed when a new message is received by the device that includes the `state.desired.hvacStatus` key-value pair. Note, you don't have to do anything to accept the desired state change; it is applied to the local **hvacStatusActuator** and its **pData** element automatically.

The `IOTUNUSED()` function is used to suppress compiler warnings for unused parameters. 

The if/else block is used to evaluate the text value of the new **hvacStatus** key-value and uses that to determine the color of the LED strips.

## Monitoring the Device Serial Output
If you closed the serial monitor from the last chapter by pressing the **CTRL** + **C** keystroke or disconnected the device, you will need to restart the serial monitor. With the device connected, run serial monitor by pasting the following command in the [PlatformIO CLI terminal window](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window):
```bash
pio run --environment core2foraws --target monitor
```

## Validation steps
Before moving on to the next chapter, you can validate that your device is configured as intended by:

1. Using the AWS IoT Core console test page, subscribe to the topic `$aws/things/<<CLIENT_ID>>/shadow/update/accepted` and you should see new messages arriving in time with your **vTaskDelay()**. (Replace <<CLIENT_ID>> with your device client Id/serial number printed on the screen.)
2. Using the AWS IoT Core console test page, publish a new shadow message on the topic `$aws/things/<<CLIENT_ID>>/shadow/update`. You should see the Core for AWS IoT EduKit's LED bars change from blue, red, none to represent the **COOLING**, **HEATING**, and **STANDBY** published values. See below for a sample shadow message. Test the effects by toggling the **hvacStatus** (set to **HEATING** or **COOLING**) and/or **roomOccupied** values (set to **true** or **false**) each time you publish the message.

```
{ "state": { "desired": { "hvacStatus": "HEATING", "roomOccupancy": true } } }
```

If these are working as expected, let's move on to [**Data transforms and routing**](/en/smart-thermostat/data-transforms-and-routing.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}