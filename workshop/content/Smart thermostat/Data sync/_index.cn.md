+++
title = "数据同步"
weight = 30
pre = "<b>c. </b>"
+++

## 章节简介
学完本章节后，您便可以使用您的 Core2 for AWS IoT Kit 参考硬件工具包执行以下操作：

* 每隔 10 秒将采样的环境声音强度和室温发布到 AWS IoT Core
* 在 AWS IoT Core 订阅您设备的影子以接收新命令

## 消息收发的概念
在本章节中，您会将设备连接到个性化的 AWS IoT Core 终端节点，然后以一种名为 “发布并订阅”（简称“PubSub”）的模式在设备和云之间交换消息。 AWS IoT Core 在很大程度上基于名为 [消息队列遥测传输](https://mqtt.org/) (MQTT) 的协议。其主页中对该协议进行了介绍：

> MQTT 是用于物联网 (IoT) 的 OASIS 标准消息协议。它是一种超轻量级的发布/订阅消息传输协议，非常适合用于连接代码量很少、网络带宽需求极低的远程设备。MQTT 如今被广泛应用于各种行业，例如汽车、制造、电信、石油和天然气等。 

在设备和云之间交换的消息可以是任何内容，包括文本、数字、二进制文件、JSON 等。JSON 是在解耦的系统之间交换消息的一种最佳实践，也是探索 AWS IoT 服务的一个推荐模式。

消息是在主题（topic）上发布的，主题是一个文本字符串，AWS IoT Core 上的其他客户端可以订阅主题以便接收被发布的消息的副本。主题可能如以下示例所示：

* `this is a topic`
* `foo/bar`
* `domain/stuff/thing/whatever`
* `dt/device123/temperature`
* `tokyo/weather/report`
* `$aws/things/edukit/shadow/update`

正如您所见，主题可能与任何文本相似。MQTT 协议将主题分区定义为由正斜杠 ‘/’ 字符分隔。这是为 IoT 解决方案设计主题架构的基本概念。本模块将帮助您以最佳实践使用主题。

在您的智能恒温器解决方案中，您将使用 AWS IoT Core 中一个名为 ”设备影子” 的功能。简单来说，设备影子是一个 JSON 文档，用于存储和检索设备的最新状态信息。您可以在 JSON 文档中存储键值对，将文档发布到特定的主题上，AWS IoT Core 将在云中存储该文档。这是一种非常有用的方法，可以将设备的最新状态更改保存在云中，以便让其他系统实时获得更新。其他系统也可以使用这种方法将期望（desired）的命令发送回设备，因为 AWS IoT Core 将使您的设备与存储在设备影子中的任何新的期望的命令保持同步。

例如，智能恒温器需要报告最新的室温和环境声音强度以支持我们的解决方案。您的设备发布的消息如下所示：

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

当 AWS IoT Core 收到类似于这样的消息时，它将存储这些键值对，直到它们被新发布的消息覆盖。

为了说明如何向您的智能恒温器发送命令，另一个系统可能向您的设备影子发布如下消息：

```JSON
{
  "state": {
    "desired": {
      "hvacStatus": "COOLING"
    }
  }
}
```

AWS IoT Core 将合并 JSON 文档，使得您的设备影子的聚合状态如下所示：

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

这意味着，当您的设备下一次连接到 AWS IoT Core 时，或者如果该设备在发布期望的命令时已连接，它将接收这种聚合状态文档。设备代码负责处理新的键值对 "hvacStatus": "COOLING"，AWS IoT Core 使设备报告新的状态值和处理期望的命令变得轻松。

如前所述，您的智能恒温器将报告最新的温度值和声音强度。您的恒温器还将接收命令并跟踪另外两个值（分别为 “hvacStatus” 和 “roomOccupancy”）的状态。这些值将在后续章节中由云应用程序确定。

## 如何以编程方式向 AWS IoT Core 发布消息
您将使用 AWS IoT Device SDK for Embedded C ("C-SDK") 在智能恒温器设备和 AWS IoT Core 之间进行通信。这是一个最佳实践，可以对安全性、网络和数据层进行抽象化，这样您便可以专注于设备和解决方案的应用程序逻辑。C-SDK 捆绑了一些库，用于通过 MQTT 协议连接到 AWS IoT Core，与硬件安全元件交互以签署请求，以及与更高阶的功能（如设备影子）集成。

我们来看一下几行重要的代码并分析它们的作用。

```C
jsonStruct_t temperatureHandler;
temperatureHandler.cb = NULL;
temperatureHandler.pKey = "temperature";
temperatureHandler.pData = &temperature;
temperatureHandler.type = SHADOW_JSON_FLOAT;
temperatureHandler.dataLength = sizeof(float);
```

上面的代码定义了一个新的 `jsonStruct_t` 类型的变量 temperatureHandler，我们将它用作一个工具来打包单独的键值对，使其能够随时在 IoT Core 设备影子中使用，以及在指定的 **pKey** 有新的 “期望” 消息到达时指明要使用的回调函数（如果有）。此示例定义了一个新的设备影子键 **temperature**，并将初始 **pData** 设置为 `temperature` 变量的值，同时将该键的类型指定为 `SHADOW_JSON_FLOAT`。稍后在注册影子 delta(增量)行为并将 “上报（reported）” 值发布到 IoT Core 时将使用这些结构体。

```C
rc = aws_iot_shadow_register_delta(&iotCoreClient, &roomOccupancyActuator);
```

上面的示例将增量行为注册到 AWS IoT 设备开发工具包。它将引用传递给初始化的开发工具包客户端和 `jsonStruct_t`，后者表示我们希望向其响应的更改的键值对。该示例注册了 **roomOccupancyActuator**，因此当客户端从云获取了一个包含新值 (`state.desired.roomOccupancy`) 的新影子更新时，系统将处理执行器中定义的回调函数。

```C
MPU6886_GetTempData(&temperature);
temperature = (temperature * 1.8)  + 32 - 50;
```

上面的示例演示了如何从 **MPU6886** 组件读取信息以获取本地温度读数。请注意到华氏温度的转换和硬编码的校准偏移量。您可以修改此表达式，使其采用摄氏温度，或者在使用默认值 50 生成的值未达到您的预期时采用其他偏移量。

```C
rc = aws_iot_shadow_add_reported(JsonDocumentBuffer,          
  sizeOfJsonDocumentBuffer, 4, &temperatureHandler,
  &soundHandler, &roomOccupancyActuator, &hvacStatusActuator);
```

此代码可将我们要发布到云的所有值都打包发送到 IoT Core 影子服务所需的影子文档。通过修改这个可变函数，您可以在此处为 `state.reported` 添加或删除键值对。不要忘记更新第三个参数以匹配列表中键值对的数量！

```C
rc = aws_iot_shadow_update(&iotCoreClient, 
  CLIENT_ID, JsonDocumentBuffer,
  ShadowUpdateStatusCallback, NULL, 4, true);
```

此代码实际上通过网络将编组的影子文档作为负载发布到 IoT Core 上的主题 `$aws/things/<<CLIENT_ID>>/shadow/update`。**<<CLIENT_ID>>** 的定义和您的 Core2 for AWS 设备的客户端 Id/序列号一致，它也显示在设备屏幕上和串行监视器输出中。

```C
while(NETWORK_ATTEMPTING_RECONNECT == rc || NETWORK_RECONNECTED == rc || SUCCESS == rc) {
...
  vTaskDelay(1000 / portTICK_RATE_MS);
}
```

除非断开网络连接，否则 **xTask** 的 `while` 循环将一直有效地运行下去。在向设备影子服务发布最新消息并响应来自收到的消息的任何增量回调后，任务将按照指定的时间进入休眠状态，然后继续进入下一个循环周期。您可以修改 `vTaskDelay()` 表达式以提高或降低发布频率。您可能会发现，在测试时使用 1000（1 秒）之类的短间隔将非常有用，在实际部署时，可将此间隔延长至 10 秒、30 秒或 60 秒。

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

此代码表示 `hvacStatus` 执行器的回调函数。此代码会在设备收到包含 `state.desired.hvacStatus` 键值对的新消息时执行。请注意，您无需执行任何操作即可接受期望的状态更改；它会自动应用于本地 **hvacStatusActuator** 及其 **pData** 元素。

`IOTUNUSED()` 函数用于禁止显示针对未使用的参数的编译器警告。

if/else 代码块用于计算新的 **hvacStatus** 键值的文本值，并使用该值来确定 LED 灯带的颜色。

## 监控设备串行输出
如果您通过按 **CTRL** + **C** 键关闭了串行显示或断开了设备，您将需要重新启动串行显示。连接好设备后，在 [PlatformIO CLI 终端窗口](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window) 中粘贴以下命令，运行串口显示：
```bash
pio run --environment core2foraws --target monitor
```

## 验证步骤
在进入下一章节之前，您可以验证您的设备是否已按预期配置完毕，方法是：

1. 使用 AWS IoT Core 控制台测试页面订阅主题 `$aws/things/<<CLIENT_ID>>/shadow/update/accepted`，您应该会看到新消息以您设置的 `vTaskDelay()` 及时到达。（使用已在屏幕上输出的您设备的客户端 Id/序列号来替换 <<CLIENT_ID>>。）
2. 使用 AWS IoT Core 控制台测试页面在主题 `$aws/things/<<CLIENT_ID>>/shadow/update` 上发布一条新影子消息。您应该会看到 Core for AWS IoT Kit 的 LED 灯条从蓝色、红色、关闭（分别表示 **COOLING**、**HEATING** 和 **STANDBY** 发布值）改变。参见下面的示例影子消息。通过在每次发布消息时切换 **hvacStatus**（设置为 **HEATING** 或 **COOLING**）和/或 **roomOccupied** 值（设置为 **true** 或 **false**）来测试效果。

```
{ "state": { "desired": { "hvacStatus": "HEATING", "roomOccupancy": true } } }
```

如果没有问题，我们就进入 [**数据转换和路由**](/cn/smart-thermostat/data-transforms-and-routing.html) 部分。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}