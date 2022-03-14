+++
title = "Conclusion"
weight = 60
pre = "<b>f. </b>"
+++

## Conclusion
You have completed this {{<awsEdukitShort-en>}} hands-on tutorial that connects your device to AWS as a smart thermostat and deploys a simple application to detect room occupancy and drive HVAC state changes. 

As a next step, consider following along with the next hands-on tutorial in this series, **Smart spaces**:

If you are looking for additional ideas to experiment with your new solution, think of ways to enhance the customer experience. For example, how could you update the solution so that the HVAC setting stays engaged for a minimum time instead of turning off after no ambient noise is heard for just 10 seconds? Can you achieve this without pushing new code to your  device? (Hint: yes, you can.)

## Clean up
In this solution you created the following resources in AWS:

* IoT Core rule.
* IAM roles.
* IoT Events input and detector model.

None of these resources incur ongoing metered charges just by existing. You will only incur metered charges as the resources are used to process new messages from your device. If you proceed with the next hands-on tutorial **Smart spaces**, we recommend that you do not destroy any of the resources from this tutorial.

If you have finished experimenting with this tutorial's solution and do not intend to explore the **Smart spaces** tutorial, you can destroy these resources: 

1. Go to the AWS IoT Core console, choose Act, choose Rules, find your rule in the list and delete it.
2. Go to the AWS IoT Events console, choose Detector models, find your model in the list and delete it. Choose Inputs, find your input in the list and delete it.
3. Go to the IAM console, choose Roles, find the roles you made for your IoT Core rules and IoT Events detector model in the list and delete them.
4. Run the command in [PlatformIO CLI terminal window](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window) from the **Smart-Thermostat** folder to erase the device firmware and prevent the device from being connected and sending messages to AWS IoT Core. You can also power down the device by holding the power button for 6 seconds:
```bash
pio run --environment core2foraws --target erase
```

The next tutorial to complete is [**Smart Spaces**](/en/smart-spaces.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}