+++
title = "Introduction"
weight = 10
pre = "<b>a. </b>"
+++

## Your task
In this tutorial, you assume the role of a full-stack developer tasked to automate the thermostat functions of a meeting room to conserve energy. To accomplish this, you use the AWS IoT EduKit reference hardware as the thermostat, or HVAC controller, and deploy an end-to-end solution that is integrated with AWS. 

The solution should recognize when employees are present in the room and engage the HVAC to deliver comfortable temperatures.The thermostat should use a narrow range of temperatures when employees occupy the room to maximize their comfort. When the room is not occupied, however, a wider range of temperatures is permitted to save energy. 
## Solution overview
Instead of using cameras to detect when the room is occupied, this solution minimizes cost by using the AWS IoT EduKit's microphone to sample noise in the room. You will capture the ambient noise level from the microphone and the room temperature from the device's sensors. Then publish these values to AWS. 

Your AWS serverless solution converts the noise level into a boolean expression (true or false) and uses that to determine whether the room is occupied. The room is considered occupied when the noise level exceeds a predefined threshold.  

While the room is occupied and the measured temperature exceeds the comfort threshold, the serverless solution sends a command to the device to begin heating or cooling. When the temperature is then measured to be within threshold, the serverless solution sends a command to the device to resume standby mode.
## Solution architecture
![Smart thermostat solution architecture](introduction/thermostat-overview.png)
{{%expand "Notes from the Smart thermostat end-to-end workflow illustration" %}}
1. The smart thermostat publishes device shadow reports. <br>
2. AWS IoT device shadow service stores the update, triggers an "update accepted" event on a new MQTT topic. <br>
3a. The topic rule evaluates the "sound" property and publishes a new device shadow for "roomOccupancy". <br>
3b. The topic rule forwards the device shadow update to AWS IoT Events for HVAC evaluation. <br>
4. Events detector model evaluates the latest "temperature" and "roomOccupancy". It then sends the new desired "hvacStatus" information to the device shadow. <br>
5. The device shadow service synchronizes updates back to the device.
{{% /expand%}}

Now that you understand the scenario and have an overview of the solution, continue to the next chapter, [**Data acquisition**](/en/smart-thermostat/data-acquisition.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}