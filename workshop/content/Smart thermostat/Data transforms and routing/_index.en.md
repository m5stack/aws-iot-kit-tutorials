+++
title = "Data transforms and routing"
weight = 40
pre = "<b>d. </b>"
+++

In this section, you transform raw data received from the AWS IoT EduKit's smart thermostat into actionable intelligence about room occupancy with application logic stored in the cloud. You also synchronize the device shadow with the state of room occupancy. Finally, you set up a resource in AWS IoT Events to receive messages from your device and forward messages published by the device's thermostat to AWS IoT Events for further processing.

## Set up the cloud solution
### Derive room occupancy from sound
The first part of your cloud solution is to add the intelligence that determines whether the room is occupied using the sound level reported by your smart thermostat device. If the sampled sound level is over a given threshold, you mark the room as occupied. If it is under the threshold, you mark the room as unoccupied. You can store this status in the device shadow and use that to sync changes back down to the device.

Why use sound level and simple threshold banding to classify room occupancy instead of a motion sensor? In this case, a microphone is the sensor that is available for use. When designing IoT solutions, you will not always have the budget for the best possible input data. This approach strikes a balance of frugality and achieving the use case. Admittedly, this will not work for every use case; such as, a team meeting where the participants use sign language, or for team meetings that start with a silent document review.

You will use two concepts of AWS IoT Core to determine room occupancy: *topic rules* and *device shadow*. A *topic rule* lets you define behavior for messages that arrive on topic filter; such as, performing live transformations on the payload and routing payloads to new destinations. The *device shadow* is a semi-structured JSON document that is used to synchronize the reported and desired state of a device. Any modifications made to your thermostat's device shadow will be sent as new payloads to your device. You have already set up your device and tested that it receives these shadow updates in the previous section.

You can combine topic rules and device shadows to make updates to the device shadow based on application logic stored in the cloud. This allows you to quickly update the application logic without pushing new code to your device. 

Your first milestone in this chapter is to create an AWS IoT topic rule that receives the messages published by your smart thermostat, inspects the sampled sound level, and updates the room occupancy state of your device shadow when it changes. The topic rule will use conditional logic in the SQL statement to construct a new payload and the IoT Core republish action to send the new payload to the device shadow. 

Complete the following:
1. Log into your AWS account.
1. Navigate to the **[AWS IoT console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/)**
1. Expand **Act** in the navigation pane. Then choose **Rules** and select **Create**.
1. Name your rule as *`thermostatRule`* and add a description. 
1. Replace **`<<CLIENT_ID>>`** in the following SQL statement with your device's client ID. 
```SQL
SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
6. Replace the contents of the *Rule query statement* field with your updated SQL statement. 
1. Choose **Add action** in the Set one or more actions section.
1. Select **Republish a message to an AWS IoT topic** and choose **Configure action** at the bottom of the window.
1. Enter **`$$aws/things/<<CLIENT_ID>>/shadow/update`** in the Topic field. Be sure to replace **`<<CLIENT_ID>>`** with your device's client ID. 
1. Choose **Create Role** in the *Choose or create a role to grant AWS IoT access to perform this action.* section. 
1. When the *Create a new role* window appears, enter **IoTRepublishRule** in the Name field and choose **Create role**.
1. Choose **Add action** to finish configuring your action and return to the rule creation form.
1. Choose **Create Rule** to create this rule in AWS IoT rules engine.

___
```SQL
SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
Let's review the SQL statement that you used in the rule a little closer. 
* The `SELECT` clause uses a `CASE` statement to achieve the threshold banding. If the sound level is over 10 (on a scale of 0-255), then treat this as an occupied room. You can modify the threshold to set your solution based on the observed degree of ambient noise.
* The `CASE` statement's output is saved to the payload key `state.desired.roomOccupancy` with the `AS` keyword. This means we are creating a new payload (like `{"state": {"desired": {"roomOccupancy": false}}}`) and sending it to the *action*.
* The `FROM` clause describes the topic filter that defines how the rule receives new messages. In this case, we want to take *action* when the device shadow service has accepted a new change; so, the statement uses `$aws/things/<<CLIENT_ID>>/shadow/update/accepted` as a topic filter. 

  There's no way to intercept the message published from the device on `$aws/things/<<CLIENT_ID>>/shadow/update` and modify it in-flight before it goes to the device shadow service because the rules engine and device shadow service receive copies of the published message in parallel.
  
  This behavior is similar to any other subscribers on the same topic when a message is published. Each subscriber receives a copy. The trade off is that for each shadow reported by the device, a second shadow update is published that computes `roomOccupancy`-- effectively doubling the traffic on the shadow. (One way to optimize this communication is to not to publish a new shadow message for `roomOccupancy` when the new value is the same as the previous one. You can achieve this by modifying the `WHERE` clause.)

* The `WHERE` clause defines a conditional statement so that the rule's actions are only executed when this statement resolves true. We use it here to prevent an infinite loop. 
  
  By including the conditional statement `state.reported.sound <> Null` this rule will only act on shadow updates where the sound value is included (or rather, not null). If the thermostat reports a `state.reported.sound` value in the shadow update, this rule will act when the thermostat publishes a message. When the rule publishes its own shadow update, the rule will not fire because the new shadow update payload includes the `state.desired.roomOccupancy` key and does not include a value for `state.reported.sound`.

* The rule's action is to "republish" (in other words, publish the output of the rule query as a new message on an indicated topic). The topic that the statement stipulates is `$$aws/things/<<CLIENT_ID>>/shadow/update` to send a new message to the device shadow. This is the same topic that your thermostat publishes to through the device shadow interface of the C-SDK.

#### Optional depth
Why are there two dollar sign characters in the topics field (`$$aws/things/<<CLIENT_ID>>/shadow/update`) but a single dollar sign in other device shadow topics? 

The reason is that the rules engine action optionally supports substitution templates. Substitution templates allow you to define an expression that is evaluated at runtime. Substitution templates use the notation `${YOUR_EXPRESSION_HERE }` and this conflicts with the `$aws` prefix of the device shadow topic. To use the correct device shadow topic in a republish action, you must escape the first dollar sign character. To do this, it requires two dollar signs: `$$aws/things/<<CLIENT_ID>>/shadow/update`.

Once you have deployed this rule in IoT Core, you should start to see the room occupancy status updated in the serial monitor output (`pio run --environment core2foraws --target monitor`).

### Prepare to determine commands for HVAC
The next milestone in this chapter is to prepare the cloud infrastructure needed to dictate new HVAC states (such as, `heating/cooling/standby`) based on current temperature and room occupancy. You will provision the AWS IoT Events service to receive messages and then create a second rule to integrate with IoT Events. This will create the data flow from IoT Core to IoT Events that is necessary before you proceed to create the detector model that dictates the HVAC state.

AWS IoT Events has two resource types: *inputs* and *detector models*. An input is a pre-defined schema to map inbound messages to detector models. A detector model is a finite state machine that processes messages from one or more inputs and determines if the state of the model should change. 

Complete these steps to create the input resource in IoT Events:
1. Navigate to the [AWS IoT Events management console](https://us-west-2.console.aws.amazon.com/iotevents/home?region=us-west-2). 
1. Expand the navigation pane. 
1. Choose **Inputs** and then choose **Create input**.
   {{< img "iot_events-create_input.webp" "Choose test in AWS IoT console" >}}
1. Enter `thermostat` into the Input name field and add a description. 
1. Create a **new file** and **save it** on your computer with the following JSON code. Name it **`input.json**.
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
6. Return to the Create input page in the AWS IoT Events console, choose **Upload file**, and select the file you created. You should see a preview of the schema interpreted from the file.
1. Choose **Create** to finish creating the input. 

Now that the input resource is created in IoT Events, you can return to IoT Core to create a new rule that will forward device shadow updates to it that you will use to create the HVAC control application. You will create the control application in the next chapter.
1. Navigate to [AWS IoT Core management console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/), expand *Act*, choose *Rules*, and choose **Create**.
1. Name your rule **sendToEvents** and add a description.
1. Replace **`<<CLIENT_ID>>`** in the following SQL statement with your device's client ID. 
```SQL
SELECT current.state as current.state, current.version as current.version, timestamp FROM '$aws/things/<<CLIENT_ID>>/shadow/update/documents'
```
4. Replace the contents of the *Rule query statement* field with your updated SQL statement. 
1. Choose **Add action**.
1. Select *Send a message to an IoT Events Input* and choose **Configure action** at the bottom of the window. 
1. Choose **Select** to the right of the *Input name* and select the *thermostatRule* input resource you created in AWS IoT Events. 
1. Choose **Create Role** in the *Role (requires IoT Events access)* section. 
1. When the *Create a new role* window appears, enter **sendToEvents** in the Name field and choose **Create role**.
1. Choose **Add action** to finish configuring your action and return to the rule creation form.
1. Choose **Create Rule** to create this rule in AWS IoT rules engine.

The rule you just created is simpler than the previous one. This rule is configured to receive the full JSON document whenever the smart thermostat device shadow is updated, and then forward it to your new IoT Events Input. The input is configured to parse only a few of the fields from the device shadow document and discard the unneeded ones.

## Validation
Before continuing to the next section, validate that your solution is configured as intended:

As your thermostat device detects different noise levels, you should see the device receive an updated status of roomOccupancy from your rule. Try playing music to make noise for ten seconds and then being quiet for ten seconds to see the state change in the serial monitor (`pio run --environment core2foraws --target monitor`).

Now that you receive updated occupancy status, continue to [**Cloud application**](/en/smart-thermostat/cloud-application.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}