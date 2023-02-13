+++
title = "构建和测试 AFI"
weight = 30
pre = "<b>c. </b>"
+++

## 章节简介
在本章节中，您将构建移植版 ESP VA-SDK 固件，并将其烧录到设备中，预置 Wi-Fi 并授权设备使用您的 Alexa 账户，以及使用此 AFI beta 版本中提供的 Alexa 语音命令测试一些智能家居功能。

## 烧录固件
使用 PlatformIO CLI 编译固件， 上传固件，并监控设备串口的输出。构建和烧录应用将会耗费一些时间，在其完成后，您应当可以在终端中看到设备日志的输出流。如果您收到了有关端口未被自动探测到的错误，参照 [在主机上识别端口](../getting-started/prerequisites/windows.html#identifying-the-device-communication-port) 中的步骤并使用其中的命令。您可以使用 **Ctrl** + **C** 组合键关闭监视的会话：
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```

## 预置设备
对于预置流程，您需要配置 Wi-Fi 网络凭证，并使用 ESP Alexa 手机 app，来授权应用程序访问您的 Alexa 账户资源。

从您的手机应用商店下载手机 app。
[iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

配置步骤：
1. 启动配套应用程序。 
   - 确保您已启用蓝牙，并且该应用程序具有访问蓝牙的适当权限。
2. 选择 **Add New Device（添加新设备）** 选项。
3. 您的设备将在应用程序中显示。
   - 您将看到一个带有值 `abcdf1234` 的 **拥有证明** 模态框或类似的弹出窗口。只需按 **Done（完成）** 即可进行下一步。
4. 连接至设备后，您将登录到您的亚马逊账户。
5. 选择 Wi-Fi 网络并输入凭证。
6. 成功连接 Wi-Fi 后，您将看到示例 utterances（表达）列表。

## 使用 Alexa
完成前面步骤后，您将会在串行监控器中看到大量的日志，包括类似下面这些内容：

```bash
I (17325) [http_transport]: Subscribing /capabilities/acknowledge...
I (17535) [http_transport]: Subscribing /connection/fromservice...
I (17735) [http_transport]: Subscribing /directive...
I (17945) [http_transport]: Subscribing /speaker...
...
I (20735) [directive_proc]: Name: EndpointForwarding
...
I (22675) [directive_proc]: Name: SetAttentionState
...
E (22685) [app_va_cb]: Enabling Mic
```

若要与 Alexa 进行互动，您需要对设备说 *Alexa*。这将触发在设备上运行的 **Espressif 唤醒词引擎**，使其进入 **侦听** 注意力状态。有关不同注意力状态的完整详情，请参阅我们的 [文档](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/ux-design-attention.html#states)。有关音频捕获的更多信息，请参阅 [SpeechRecognizer API 文档](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/avs-speechrecognizer-concepts.html)。

{{< img "speechrecognizer-state.en.png" "Audio Capture Speech Recognizer Attention States" >}} 

{{% notice info %}}
就像其他 Alexa 设备一样，当该设备处于 “空闲” 状态时，它只侦听关键词 “Alexa”。只有在触发关键唤醒词后，设备才会开始将音频上传到云。
{{% /notice %}}

试着对 Alexa 说不同的表达 - 当设备听到 **Alexa** 时，侧面的 LED 灯应亮起蓝色的光（如果 Alexa 没有 “醒来”，试着靠近设备说话）：
* _Alexa, what time is it?_
* _Alexa, tell me a joke?_
* _Alexa, turn on all of the lights_ (仅仅在您的同一个账户内已经有了一些 Alexa 智能家居设备时）

{{< img "alexa-time.en.webp" "Alexa, what time is it?" "Person: Alexa, what time is?, Alexa: It's 1:52p.m.">}}

## 测试 Alexa 智能家居功能 (Beta)
AFI 设备具备 **Alexa Built-in**，这意味着您可以直接在设备上与 Alexa 对话，Alexa 会在设备上通过语音进行回应。但是，Espressif 的 AFI 版本还支持 Alexa 智能家居命令（作为 beta 版功能），这可以让您控制设备上的属性。

Alexa for AWS IoT 示例应用程序会在您的 Alexa 应用程序中创建一个名为 **Light** 的虚拟设备，该设备支持两个接口：

* [PowerController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-powercontroller.html)： 开灯和关灯。
* [RangeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-rangecontroller.html)：调整设备的亮度。

![名为 "Light" 的设备应该会展示在你的 Alexa App内](building-and-testing-afi/alexa_app-light_device.en.png?height=500px)

因为它是虚拟设备，所以它会将更新的状态打印到屏幕上。我们可以通过语音或 Alexa 应用程序进行测试。

* 通过语音 - 说 _Alexa, turn on the light_ - 如果成功，Alexa 可能会回复 "OK" 或其他确认声音。
* 通过 Alexa 应用程序 - 打开您的 Alexa 手机 app (并不是 ESP Alexa 手机 app)，转到设备，然后转到 **Lights（灯）** 或 **All Device（所有设备）**，您应该能看到名为 **Light** 的设备（参见屏幕截图）。点击电源图标，您应该会看到开关切换。

不管使用哪种方式，您都会在终端中看到类似以下内容的消息：
```bash
I (97445) [alexa_smart_home]: Namespace: Alexa.PowerController, Name: TurnOn
```

同样，您可以尝试通过以下方式之一控制亮度：

* 通过语音 - 说 _Alexa, set brightness on the light to 80_ - 如果设置成功，Alexa 可能会回复 “OK” 或使用其他确认回复。
* 通过 Alexa 应用程序 - 调整亮度滑块。

不管使用哪种方式，您都会在串行监控器中看到类似以下内容的消息：
```bash
[app_smart_home]: *************** Light's Brightness changed to 60 ***************
```

这非常有用，不仅因为我们的设备上有 Alexa 的语音助手，还因为我们可以使用 Alexa 来控制设备本身的属性！

接下来，我们将创建 [**自定义智能家居设备**](/cn/intro-to-alexa-for-iot/custom-smart-home-device.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}