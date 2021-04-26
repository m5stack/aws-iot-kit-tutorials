+++
title = "Building and Testing AFI"
weight = 30
pre = "<b>c. </b>"
+++

## Chapter introduction
In this chapter we will build the ported ESP VA-SDK firmware, flash it on to the device, provision the Wi-Fi and authorize the device to your Alexa Account, and test some of the smart home capabilities using Alexa voice commands available in this beta version of AFI.

## Flash the Firmware
Use the PlatformIO CLI to compile your firmware, upload the firmware, and monitor the serial output of your device. It will take some time to build and flash the app, but after that's done you should see the stream of device logs in your terminal. If you receive an error for the port not being auto-detected, follow the [Identifying the serial port on host machine](../getting-started/prerequisites/windows.html#identifying-the-device-communication-port) instructions and try the command again. You can close the monitor session with the **Ctrl** + **C** keystroke combination:
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```

## Provision the Device
For the provisioning process, you will need to configure your Wi-Fi network credentials and authorize the application with your Alexa account using ESP Alexa Phone Application.

Download the application from your mobile app store.
[iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

Provision steps:
1. Launch the companion app.
   - Make sure you have enabled the Bluetooth and the app has proper permissions to access Bluetooth. 
2. Select the option **Add New Device**.
3. Your device will display in the app. 
   - You will see a **Proof of Possession** modal with a value of `abcdf1234` or similar pop up. Simply press **Done** to continue.
4. After connecting to the device, you will sign into your Amazon account. 
5. Select the Wi-Fi network and enter the credentials.
6. After a successful Wi-Fi connection, you will see a list of sample utterances.

## Using Alexa
With the prior steps completed, you will see a number of logs in your serial monitor, including some like the following:

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

In order to interact with Alexa, you will need to say *Alexa* to the device. This will trigger the **Espressif Wake Word Engine** running on the device to enter the **LISTENING** attention state. For full details on the different attention states, please refer to our [documentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/ux-design-attention.html#states). For more information on audio capture, see the [SpeechRecognizer API Documentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/avs-speechrecognizer-concepts.html)

{{< img "speechrecognizer-state.en.png" "Audio Capture Speech Recognizer Attention States" >}} 

{{% notice info %}}
Just like any Alexa device, when the device is in the IDLE state, it is listening ONLY for the keyword "Alexa". Only once the keyword is triggered will the device start streaming audio to the cloud.
{{% /notice %}}

Try a variety of utterances to Alexa - the side LEDs should light blue up when **Alexa** is heard (if Alexa does not "wake up", try speaking closer to the device):
* _Alexa, what time is it?_
* _Alexa, tell me a joke?_
* _Alexa, turn on all of the lights_ (Only works if you already have some Alexa smart home devices on your same account)

{{< img "alexa-time.en.webp" "Alexa, what time is it?">}}

## Testing Alexa Smart Home Capabilities (Beta)
The AFI device has **Alexa Built-In**, which means you can speak to Alexa directly to the device and Alexa will respond with voice on the device. However, this version of AFI from Espressif also supports Alexa Smart Home commands as a beta feature, which allows you to control attributes on the device.

The Alexa for AWS IoT sample application creates a virtual device called **Light** in your Alexa app, which supports two interfaces:

* [PowerController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-powercontroller.html) to turn the light on and off.
* [RangeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-rangecontroller.html) to adjust the brightness of the device.

![The device named "Light" should show up in your Alexa App](building-and-testing-afi/alexa_app-light_Device.en.png?height=500px)

Since it's a virtual device, it is prints the updated status to the screen. We can test this out via voice or the Alexa app.

* By voice - say _Alexa, turn on the light_ - if successful, Alexa may respond with "OK" or some other confirmation sound
* Via the Alexa app - open your Alexa mobile phone app (not the ESP Alexa mobile phone app), go to Devices and then either **Lights** or **All Devices** and you should see the device named **Light** (see screenshots below). Tap the power icon and you should see the icon toggle between off and on. 

Through either option, you should see a message like the following in your terminal:
```bash
I (97445) [alexa_smart_home]: Namespace: Alexa.PowerController, Name: TurnOn
```

Similarly, you can try to control the range of the brightness by one of the following: 

* By voice - say _Alexa, set brightness on the light to 80_ - if successful, Alexa may respond with "OK" or some other confirmation response.
* Via the Alexa app - adjust the slider of brightness.

Through either option, you should see a message like the following in the serial monitor:
```bash
[app_smart_home]: *************** Light's Brightness changed to 60 ***************
```

This is useful not only because we have voice assistant on our device with Alexa, but we can use Alexa to control properties on the device itself!

On to creating a [**Custom Smart Home Device**](/en/intro-to-alexa-for-iot/custom-smart-home-device.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}