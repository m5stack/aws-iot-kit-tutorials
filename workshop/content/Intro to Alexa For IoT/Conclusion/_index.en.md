
+++
title = "Conclusion"
weight = 50
pre = "<b>e. </b>"
+++

## Conclusion
You have completed the {{< awsService type="edukit-short-en" >}} hands-on tutorial for running Alexa for AWS IoT (AFI) using the custom ported Espressif Voice Assistant SDK and Alexa to control on-board peripherals using Alexa Smart Home commands. For additional information about Smart Home for AVS, see the [documentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/smart-home-for-avs.html) or the following [blog post](https://developer.amazon.com/en-US/blogs/alexa/device-makers/2020/04/create-a-sample-alexa-built-in-disco-ball-with-smart-home-for-av).

If you would like to learn more about other interfaces available or come up with your own implementation using other features of the {{< awsService type="edukit-long-en" >}}, take a look at [ModeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-modecontroller.html) and [ToggleController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-togglecontroller.html). Use your creativity and what you've learned in the other chapters of this tutorial to explore new capabilities for voice! 

This is the last tutorial currently available. We will be publishing more, but for now, you are able to build your own IoT solution with AWS services using the skills you have learned so far.

## Clean up
In this solution you did not use any of your own resources in AWS. However, if you are done using the Alexa application, you can completely wipe your device's flash memory (and remove the certificates from flash) by entering this command in the [PIO CLI terminal window](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window):
```bash
pio run --environment core2foraws --target erase
```

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}