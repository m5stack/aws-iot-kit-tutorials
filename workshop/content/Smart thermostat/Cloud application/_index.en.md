+++
title = "Cloud application"
weight = 50
pre = "<b>e. </b>"
+++

## Chapter introduction
By the end of this chapter, your solution will do the following:

* Process telemetry messages from the Core2 for AWS IoT EduKit device in a managed cloud application
* Determine which state the HVAC system should be in: HEATING, COOLING, or STANDBY
* Send a desired state message to your device shadow based on processed inputs
* Sync the desired shadow state to your device
* Update the device display with the desired HVAC system state

## How to build the cloud application
In this chapter you will construct a serverless application without any code that analyzes the inputs from your smart thermostat and determines the behavior for the corresponding (fictitious) HVAC system. You will use IoT Events to deploy a resource called a detector model that processes the device shadow messages forwarded by your IoT Core rule, evaluate if any state changes should occur from heating to cooling to standby, then send a message back to your smart thermostat with the updated state change, if any.

Here is a preview of the detector model to create:
{{< img "iot-events-detector_model.en.png" "Detector model" >}}

As you can see, there are three states for the HVAC application: heating, cooling, and standby. The application will always be in one of those states and initializes to the *standby* state. As new messages arrive from the smart thermostat, the detector model evaluates the inputs against conditional logic to determine if the model should transition to a new state.

For example, from the *standby* state, if a new temperature reading comes in over 80 (in degrees Fahrenheit), the model will evaluate a transition to the *cooling* state. Then upon entering the *cooling* state, the model will publish a new message back to IoT Core to update the device shadow of the smart thermostat with a message like: `{ "state": { "desired": { "hvacStatus": "COOLING" } } }`. Based on the work you initially did in the chapter **Data sync**, these new commands on the desired node will be acknowledged and the state on the device updated to reflect the command by updating the LED strips to be red, blue, or off.

It is beyond the scope of this learning module to fully explore how to create detector models in IoT Events. Instead, below are steps for getting your detector model up and running, with a few notes on key pieces so you know how _this_ model works. For a primer on getting started with AWS IoT Events, please review the [developer guide](https://docs.aws.amazon.com/iotevents/latest/developerguide/what-is-iotevents.html).

To create the detector model previewed in the image above, complete the following steps.

First, you will create a file on your local desktop with the following contents. Name the file something like `model.json`. You must edit the file to find and replace the three instances of the term **<<CLIENT_ID>>** with the client Id/serial number of your device (it is printed on your device screen).

```json
{
    "detectorModelDefinition": {
        "states": [
            {
                "stateName": "standby",
                "onInput": {
                    "events": [],
                    "transitionEvents": [
                        {
                            "eventName": "startHeating",
                            "condition": "($input.thermostat.current.state.reported.temperature <= $variable.heatingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature <= $variable.heatingThresholdOccupied) ",
                            "actions": [],
                            "nextState": "heating"
                        },
                        {
                            "eventName": "startCooling",
                            "condition": "($input.thermostat.current.state.reported.temperature >= $variable.coolingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature >= $variable.coolingThresholdOccupied) ",
                            "actions": [],
                            "nextState": "cooling"
                        }
                    ]
                },
                "onEnter": {
                    "events": [
                        {
                            "eventName": "setThresholds",
                            "condition": "true",
                            "actions": [
                                {
                                    "setVariable": {
                                        "variableName": "heatingThresholdUnoccupied",
                                        "value": "60"
                                    }
                                },
                                {
                                    "setVariable": {
                                        "variableName": "heatingThresholdOccupied",
                                        "value": "68"
                                    }
                                },
                                {
                                    "setVariable": {
                                        "variableName": "coolingThresholdOccupied",
                                        "value": "72"
                                    }
                                },
                                {
                                    "setVariable": {
                                        "variableName": "coolingThresholdUnoccupied",
                                        "value": "80"
                                    }
                                }
                            ]
                        },
                        {
                            "eventName": "setStandbyInShadow",
                            "condition": "true",
                            "actions": [
                                {
                                    "iotTopicPublish": {
                                        "mqttTopic": "'$aws/things/<<CLIENT_ID>>/shadow/update'",
                                        "payload": {
                                            "contentExpression": "'{\"state\":{\"desired\":{\"hvacStatus\":\"STANDBY\"}}}'",
                                            "type": "JSON"
                                        }
                                    }
                                }
                            ]
                        }
                    ]
                },
                "onExit": {
                    "events": []
                }
            },
            {
                "stateName": "cooling",
                "onInput": {
                    "events": [],
                    "transitionEvents": [
                        {
                            "eventName": "stopCooling",
                            "condition": "(!$input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature < $variable.coolingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature < $variable.coolingThresholdOccupied) ",
                            "actions": [],
                            "nextState": "standby"
                        }
                    ]
                },
                "onEnter": {
                    "events": [
                        {
                            "eventName": "setCoolingInShadow",
                            "condition": "true",
                            "actions": [
                                {
                                    "iotTopicPublish": {
                                        "mqttTopic": "'$aws/things/<<CLIENT_ID>>/shadow/update'",
                                        "payload": {
                                            "contentExpression": "'{\"state\":{\"desired\":{\"hvacStatus\":\"COOLING\"}}}'",
                                            "type": "JSON"
                                        }
                                    }
                                }
                            ]
                        }
                    ]
                },
                "onExit": {
                    "events": []
                }
            },
            {
                "stateName": "heating",
                "onInput": {
                    "events": [],
                    "transitionEvents": [
                        {
                            "eventName": "stopHeating",
                            "condition": "(!$input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdOccupied) ",
                            "actions": [],
                            "nextState": "standby"
                        }
                    ]
                },
                "onEnter": {
                    "events": [
                        {
                            "eventName": "setHeatingInShadow",
                            "condition": "true",
                            "actions": [
                                {
                                    "iotTopicPublish": {
                                        "mqttTopic": "'$aws/things/<<CLIENT_ID>>/shadow/update'",
                                        "payload": {
                                            "contentExpression": "'{\"state\":{\"desired\":{\"hvacStatus\":\"HEATING\"}}}'",
                                            "type": "JSON"
                                        }
                                    }
                                }
                            ]
                        }
                    ]
                },
                "onExit": {
                    "events": []
                }
            }
        ],
        "initialStateName": "standby"
    },
    "detectorModelName": "hvacApplication",
    "detectorModelDescription": "cloud application to manage HVAC state",
    "evaluationMethod": "BATCH"
}
```

Next, you will use this file in the steps to import a new detector model in the AWS IoT Events console.
1. Go to the AWS IoT Events management console.
2. Choose **Detector models**.
3. Choose **Action** and then **Import detector model**.
4. Choose **Import**.
5. Find the file you created in the previous step and choose **Open** (button name may vary per browser).
6. Choose **Publish**.
7. Under *Role*, provide a new name that will be used to create a new IAM role, like `edukit-iotevents`. The console will create a new IAM role that gives IoT Events permission to operate your model.
8. Choose **Save and publish**.

You have now deployed the detector model to IoT Events. As new messages are published from your smart thermostat, the IoT Core rule you created in the previous chapter forwards them to the IoT Events input resource, then IoT Events pushes copies of the messages from any inputs to any consuming detector models, like this one, for evaluation of any state changes.

Now for an explanation of a few key pieces of the model. Each state of the model (heating/cooling/standby) is nearly the same. The *standby* state has one extra action to set the numeric thresholds that every state uses to determine whether it should change states. These could be set once in a one-time initialization state before moving to *standby* for the first time, but they are included in *standby* for simplicity of the demonstration.

The rest of the states' configuration is similar. There is an OnEnter event to publish a message to the smart thermostat's device shadow, indicating which state the HVAC should now be in. There is a conditional transition that gets evaluated after each message is processed that checks if the state should change. For example, if the model is in the *heating* state and the conditional expression for *stopHeating* evalutes true, the model will move to the *standby* state. Note that the *standby* state has two transitions, one each for moving to *heating* or *cooling*. This model is simplified in that the system will always go through the *standby* state instead of directly from *heating* to *cooling* or vice versa. The conditional transition logic takes into consideration the two temperature boundaries; one when the room is occupied and another when the room is unoccupied.

Below is a sample transition from the *heating* state labeled *stopHeating*. There are two boolean expressions joined with OR `||` logic. In plainer English, this expression means "stop heating if the room is unoccupied and the room temperature is warm enough while unoccupied, OR if the room is occupied and the room is warm enough for people."

```js
(!$input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdOccupied)
```

If everything has been configured per this module's instructions, you should now be seeing status changes delivered to your smart thermostat in the form of updating LED strips, completing the end-to-end solution!

## Validation steps
Before moving on to the next chapter, you can validate that the solution is configured as intended by:

1. Apply a temperature change to your device that will take it outside the comfort bounds set in the IoT Events detector model to see the LED strips on the side change to red (heating), blue (cooling), or off (for standby). Use the warmth of your hands or a fan to raise or lower the detected temperature.
1. Alternatively, choose new temperature thresholds for your event detector that exclude your room's true ambient temperature and redeploy your detector model to see it work. You can find the threshold values stored as variables in the model state labeled "standby" under the OnEnter action labeled "setThresholds." You can update those numbers to anything that makes sense for your room and publish the changes to test. 

Note: the temperature sensor is inside the housing of the kit and the code includes a hardcoded offset. You may see variance in the temperature reading up to 10 degrees from reading to reading. 

If the solution is working as expected, you can move on to the [**Conclusion**](/en/smart-thermostat/conclusion.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}