+++
title = "Data transforms and routing"
weight = 40
pre = "<b>d. </b>"
+++

## Chapter introduction
By the end of this chapter, your cloud solution should do the following:

* Transform raw sound data received from the device into actionable intelligence of room occupancy with application logic stored in the cloud
* Synchronize the state of room occupancy from the device shadow down to the device
* Set up a resource in AWS IoT Events to receive messages from your device
* Forward messages published by the thermostat device to AWS IoT Events for further processing 

## How to set up the cloud solution
### Deriving room occupancy from sound
The first part of your cloud solution is to add the intelligence that derives whether the room is occupied by people using it. To approximate room occupation, you will use the sound level reported by the smart thermostat device. If the sampled sound level is over a given threshold, you will mark the room as occupied. If it is under the threshold, you will mark the room as unoccupied. You can store this status in the device shadow and use that to sync changes back down to the device.

Why use sound level and simple threshold banding to classify room occupancy? Why not use a motion sensor? In this case, a microphone is the sensor that is available for use. When designing IoT solutions, you will not always have the budget for the best possible input data. This approach strikes a balance of frugality and achieving the use case. The author acknowledges this would not work for every use case, such as a team meeting for people who are hard of hearing and using sign language, or for team meetings starting with a silent document review.

You will use two concepts of AWS IoT Core to achieve this use case of deriving room occupancy. These are *topic rules* and *device shadow*. A topic rule lets you define behavior for messages arriving on topic filter, such as performing live transformations on the payload and routing payloads to new destinations. The device shadow is a semi-structured JSON document for synchronizing the reported and desired state of a device. Any modifications made to the device shadow of your thermostat will be sent as new payloads to your device. You have already set up and tested that your device is receiving these shadow updates in the previous chapter.

You can combine topic rules and device shadows to functionally make updates to the device shadow based on application logic stored in the cloud. This enables rapid updating of application logic without pushing new code to your device. 

Your first milestone in this chapter is to create an IoT Core topic rule that receives the messages published by your smart thermostat, inspects the sampled sound level, and updates the room occupancy state of your device shadow as it changes. The topic rule will use conditional logic in the SQL query to construct a new payload and the IoT Core republish action to send the new payload to the device shadow. 

1. Go to the AWS IoT Core management console, choose *Act*, choose *Rules*, and choose **Create**.
2. Give your rule a name and description. Further steps in this material assume the name is `thermostatRule`.
3. Use the following query and be sure to replace **<<CLIENT_ID>>** with your device client Id/serial number.
```SQL
SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
4. Choose **Add action**.
5. Select *Republish a message to an AWS IoT topic* and choose **Configure action**.
6. For *Topic*, use `$$aws/things/<<CLIENT_ID>>/shadow/update`. Be sure to replace **<<CLIENT_ID>>** with your device's client Id/serial number.
7. For *Choose or create a role to grant AWS IoT access to perform this action.* choose **Create Role** and in the pop-up give your new IAM role a name, then choose **Create role**.
8. Choose **Add action** to finish configuring your action and return to the rule creation form.
9. Click **Create Rule** to create this rule in AWS IoT rules engine.

Let's break down this rule and explain the parts. The SELECT clause uses a CASE statement to achieve our simple threshold banding. If the sound level reported by the device is over 10 (on a scale of 0-255), then we treat this as an occupied room. You can modify the 10 to set your solution's threshold for an occupied room based on the observed degree of ambient noise.

The output of the CASE statement is saved to payload key `state.desired.roomOccupancy` with the AS keyword. This means we are creating a new payload like `{"state": {"desired": {"roomOccupancy": false}}}` and sending this payload on to the action.

The FROM clause describes the topic filter on which this rule will receive new messages. In this case, we want to take action when the device shadow service has accepted a new change, so we use the topic filter `$aws/things/<<CLIENT_ID>>/shadow/update/accepted`. There's no way to intercept the message published from the device on `$aws/things/<<CLIENT_ID>>/shadow/update` and modify it in-flight before it goes to the device shadow service because the rules engine and device shadow service receive copies of the published message in parallel. This behavior is just like any other two subscribers on the same topic when a message is published; they will each receive a copy. The trade off of this pattern is that for each shadow reported by the device, a second shadow update is published that computes the roomOccupancy, effectively doubling the traffic on the shadow. (One way to optimize here is not to publish a new shadow message for roomOccupancy if the new value is the same as the previous one. This can be achieved by modifying the WHERE clause. This educational solution prefers simplicity over optimization.)

The WHERE clause defines a conditional statement such that the rule's actions are only executed if this statement resolves true. We use it here to prevent an infinite loop. By including the conditional statement `state.reported.sound <> Null` we are configuring this rule to only act on shadow updates where the sound value is included (or rather, not null). The thermostat reports a `state.reported.sound` value in the shadow update, so this rule will act when the thermostat publishes messages. When the rule publishes its own shadow update, the rule will not fire again because the new shadow update payload only includes the `state.desired.roomOccupancy` key and no value for `state.reported.sound`.

The action of this rule is "republish" or in other words, publish the output of the rule query as a new message on an indicated topic. The publish topic we use in this action is `$$aws/things/<<CLIENT_ID>>/shadow/update` to send a new message to the device shadow. This is the same topic that your thermostat is publishing on via the device shadow interface of the C-SDK.

(Optional depth) Why do we use two $ characters here but a single $ in other uses of the device shadow topics? The reason is the rules engine action optionally supports substitution templates. Substitution templates let you define an expression that gets evaluated at runtime. Substitution templates use the notation `${ YOUR_EXPRESSION_HERE }` and this conflicts with the `$aws` prefix of the device shadow topic. To use the correct device shadow topic in a republish action, you must escape the first $ character and this looks like `$$aws/things/<<CLIENT_ID>>/shadow/update`.

Once you have deployed this rule in IoT Core, you should start to see the room occupancy status updated in the serial monitor output (`pio run --environment core2foraws --target monitor`).

### Preparing to determine commands for HVAC
The next milestone in this chapter is to prepare the cloud infrastructure needed to dictate new HVAC states (e.g. heating/cooling/standby) based on current temperature and room occupancy. You will provision the IoT Events service to receive messages and then create a second IoT Core rule to integrate with IoT Events. This will create the data flow from IoT Core to IoT Events that is necessary before you move on to create the detector model that dictates the HVAC state.

IoT Events has two resource types: inputs and detector models. An input is a pre-defined schema for mapping inbound messages to detector models. A detector model is a finite state machine that processes messages from one or more inputs and determines if the state of the model should change. 

Follow these steps to create the input resource in IoT Events:
1. Go to [IoT Events management console](https://us-west-2.console.aws.amazon.com/iotevents/home?region=us-west-2). Expand the left menu, choose **Inputs**, then choose **Create input**.
   {{< img "iot_events-create_input.webp" "Choose test in AWS IoT console" >}}
2. Name your input `thermostat` and give it a description. Further steps in this module are dependent upon the name being `thermostat`.
3. You must upload a JSON file to define the schema. Create a new file on your computer with the following contents and a file name like `input.json`:
```JSON
{
  "current": {
    "state": {
      "reported": {
        "sound": 10,
        "temperature": 35,
        "roomOccupancy": false,
        "hvacStatus": "HEATING"
      }
    },
    "version": 13
  },
  "timestamp": 1606282489
}
```
4. Choose **Upload file** and select the new file you created. You should see a preview of the schema interpreted from the file.
5. Choose **Create** to finish creating the input. 

Now that the input resource is created in IoT Events, you can return to IoT Core to create a new rule that will forward device shadow updates to it that you will use to create the HVAC control application. You will create the control application in the next chapter.
1. Go back to [AWS IoT Core management console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/), choose *Act*, choose *Rules* and choose **Create**.
2. Give your rule a name and description.
3. Use this statement for the query and be sure to replace **<<CLIENT_ID>>** with your device's client Id/serial number: 
```SQL
SELECT current.state as current.state, current.version as current.version, timestamp FROM '$aws/things/CLIENT_ID/shadow/update/documents'
```
4. Choose **Add action**.
5. Select *Send a message to an IoT Events Input* and choose **Configure action**.
6. For *Input name*, find the name of the Input resource you created in the previous step of the IoT Events console. The assumed name is `thermostat`.
7. For *Role*, choose **Create Role**, give your role a name, then choose **Create role** to finalize your new IAM role that gives permission to IoT Core to send messages to IoT Events.
8. Choose **Add action** to finalize your new rule action. You should be returned to the rule creation form.
9. Choose **Create rule** to finish creating the rule.

This rule is much simpler compared to the previous one! The rule is configured to receive the full JSON document whenever the smart thermostat device shadow is updated, then forward it to your new IoT Events Input. The Input is configured to parse only a few of the fields from the device shadow document and will discard the unneeded ones.

## Validation steps
Before moving on to the next chapter, you can validate that your solution is configured as intended by:

1. As your thermostat device detects varying noise levels, you should see the device receive updated status of roomOccupancy from your rule. Try alternating playing some music to make noise for ten seconds and being quiet for ten seconds to see the state change in the serial monitor (`pio run --environment core2foraws --target monitor`).

If these are working as expected, let's move on to [**Cloud application**](/en/smart-thermostat/cloud-application.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}