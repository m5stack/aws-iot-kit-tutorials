+++
title = "Introduction"
weight = 10
pre = "<b>a. </b>"
+++

## Your task
In this scenario, you are taking on the role of a full-stack developer tasked with automating thermostat functions of a meeting room in order to conserve energy. You will use the Core2 for AWS IoT EduKit reference hardware for the thermostat hardware and deploy an end-to-end solution combining the reference hardware kit as an HVAC controller with the power of the AWS cloud. We assume a fictional HVAC system and terminate the solution at the edge with your Core2 for AWS IoT EduKit as a thermostat.

The thermostat should use a narrower range of temperatures when employees are occupying the room to maximize their comfort. When the room is not occupied, a wider range of temperatures is permitted to save energy. The solution should recognize when employees are present in the room and engage the HVAC to deliver the comfort zone of temperature.

## Problem solving
To minimize costs of needing camera equipment to detect when the room is occupied, this solution will use the kit microphone to sample a simple noise level. You will capture the ambient noise level from the microphone and the room temperature from the device's sensors, then publish these values to AWS. 

Your serverless solution in AWS will convert noise levels into a boolean and use that to determine whether the room is occupied. You can use a simple threshold of noise level to make that determination. 

While the room is occupied and the measured temperature is out of comfort bounds, the solution will send a command to the device to start heating or cooling, as needed. When the temperature is then measured as within bounds, the solution will send a command to the device to resume a standby mode.

## Solution architecture
![Smart thermostat solution architecture](introduction/thermostat-overview.png)

## Let's go!
Are you ready to start building? Let's review you have the following prerequisites sorted:

1. Have you completed the **Blinky Hello World** tutorial? 
2. Is your Core2 for AWS IoT EduKit already provisioned in AWS IoT Core? Meaning there is a registered AWS IoT thing with a certificate and policy attached that enables publish and subscribe operations? These steps are in the **Blinky Hello World** tutorial.
3. Have you confirmed that you can see messages arriving from your device using a test MQTT client like the one in the AWS IoT Core console? You should be able to subscribe to a topic that your device is publishing on and see those messages arrive in the test client.
4. Do you know which serial port your Core2 for AWS IoT EduKit device is mounted to? This is also covered in the **Blinky Hello World** tutorial.

If so, let's begin by moving on to the next chapter, [**Data acquisition**](/en/smart-thermostat/data-acquisition.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}