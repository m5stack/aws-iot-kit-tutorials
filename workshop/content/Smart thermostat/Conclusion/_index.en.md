+++
title = "Conclusion"
weight = 60
pre = "<b>f. </b>"
+++

We hope that you enjoyed connecting your {{< awsService type="edukit-short-en" >}} to AWS, and deploying a simple application that detected room occupancy and controlled a fictitious HVAC system. 

If you would like additional ideas for experimentation, think of ways to enhance the customer experience. For example, how could you update the solution so that the HVAC settings stay engaged for a minimum of time, instead of turning off after 10 seconds when no ambient noise is detected? How could you achieve this without pushing new code to your device? *(You can do it!)*


## Clean up
In this solution, you created the following resources in AWS:

* AWS IoT rule
* IAM roles
* IoT Events input and detector model

None of these resources incur ongoing metered charges just by existing. You only incur charges when the resources are used to process new messages from your device. If you proceed with the next hands-on tutorial [**Smart Spaces**](/en/smart-spaces.html), do not remove any of the resources from this tutorial. If you will take a break between tutorials, simply turn off your device (hold the power button for six seconds) to prevent it from continuing to publish messages to AWS.

If you have finished experimenting with this tutorial's solution and do not intend to explore the [**Smart Spaces**](/en/smart-spaces.html) tutorial, you can remove these resources: 

1. Navigate to [AWS IoT Core](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/home), expand Act, choose Rules, find your rule in the list and delete it.
1. Navigate to [AWS IoT Events](https://us-west-2.console.aws.amazon.com/iotevents/home?region=us-west-2), choose Detector models, find your model in the list and delete it. 
1. Continue in AWS IoT Events, choose Inputs, find your input in the list and delete it.
1. Navigate to [AWS IAM](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/home) console, choose Roles, find the roles you made for your IoT Core rules and IoT Events detector, and delete them.
1. Connect your {{< awsService type="edukit-short-en" >}} to your host machine and make sure that it's on. 
1. Open the Smart-Thermostat folder in VS Code. 
1. Open a [PlatformIO terminal window](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window) and issue the following command to erase the device's firmware.
```bash
pio run --environment core2foraws --target erase
```



Please consider continuing on your journey with your {{< awsService type="edukit-short-en" >}}. On to [**Smart Spaces**](/en/smart-spaces.html)!

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}