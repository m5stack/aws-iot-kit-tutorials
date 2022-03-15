+++
title = "Cloud application"
weight = 50
pre = "<b>e. </b>"
+++

## Chapter introduction
In this lesson, you process telemetry messages from the {{<awsEdukit-short-en>}} in a managed cloud application, determine what state to set the HVAC system (`HEATING`, `COOLING`, or `STANDBY`.), and send a *desired state* message to your device shadow based on the processed inputs. You then sync the desired shadow state to your device to update the device display with the desired HVAC system state.

## How to build the cloud application
To accomplish these goals, construct a serverless application that analyzes the inputs from your smart thermostat, and determines the behavior for the corresponding, and fictitious, HVAC system. Use IoT Events to deploy a resource, called a detector model, to process the device shadow messages that are forwarded by your AWS IoT rule. The detector model evaluates if any state changes should occur from heating to cooling to standby, then sends a message back to your smart thermostat with the updated state change, as necessary.

Here is a preview of the detector model you will create:
{{< img "iot-events-detector_model.en.png" "Detector model" >}}

The mode has three states for the HVAC application: heating, cooling, and standby. The application is always in one of those states and begins in the `STANDBY` state. As new messages arrive from the smart thermostat, the detector model evaluates these inputs against conditional logic to determine if the model should transition to a new state.

For example, from the `STANDBY` state, if a new temperature reading comes in over 80 (in degrees Fahrenheit), the model will evaluate a transition to the `COOLING` state. Then, when it enters the `COOLING` state, the model publishes a new message to IoT Core to update the smart thermostat's device shadow. It's message will be similar to: `{ "state": { "desired": { "hvacStatus": "COOLING" } } }`. 

Based on the work you performed in the [Data sync](/en/smart-thermostat/data-sync.html) lesson, new commands are acknowledged, the state on the device is updated to reflect the command, and the LED strips are set to red, blue, or off.

It is beyond the scope of this lesson to fully explore how to create detector models in IoT Events. Instead, complete the following steps to create your detector model. For more information about detector models and AWS IoT Events, see [What is AWS IoT Events?](https://docs.aws.amazon.com/iotevents/latest/developerguide/what-is-iotevents.html).

1. Create a file on your host machine named **`model.json`**.
1. Copy and paste **the following code** into the document.

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
3. Edit the document to replace **<<CLIENT_ID>>** (**there are three instances**) with **your device's client ID** (the ID displays on the {{<awsEdukit-short-en>}}'s screen).
1. Log into [AWS](https://aws.amazon.com/), if necessary. 
1. Navigate to the [AWS IoT Events](https://us-west-2.console.aws.amazon.com/iotevents/home?region=us-west-2#/) console.
1. Expand the **navigation panel**, if necessary, and choose **Detector models**.
1. Expand the **Action** menu and select **Import detector model**.
1. Choose **Import** in the confirmation window.
1. Find the **file you created** in the previous step and choose **Open**.
1. Choose **Publish**.
1. Enter **`edukit-iotevents`** in the Role field to create a new IAM role grants IoT Events permission to operate your model.
1. Choose **Save and publish**.

    After a few minutes, you should see status changes delivered to your smart thermostat through the LED strips color changes.

**Congratulations!** You deployed a  detector model to IoT Events. As new messages publish from your smart thermostat, the IoT Core rule you created in the previous chapter forwards them to the IoT Events input resource. IoT Events then pushes copies of the messages from the inputs to detector models that consume them, like this one, so it can evaluate any state changes.

Before you continue with this lesson, let's pause to review a few key pieces of the detector model. 
* Each state of the model (`HEATING`/`COOLING`/`STANDBY`) is nearly the same. The `STANDBY` state has an additional action that sets the numeric thresholds for the other state (`HEATING` and `COOLING`). Alternately, the detector could have been configured to set the thresholds during a one-time initialization state.

* The configuration for the other states is similar:
    * There is an `OnEnter` event to publish a message to the smart thermostat's device shadow that indicates which state the HVAC should be in. 
    * There is a conditional transition that is evaluated after each message processes and checks if the state should change; for example, if the model is in the `HEATING` state and the conditional expression for `stopHeating` evaluates `true`, the model transitions to `STANDBY`. **Note:** `STANDBY` has two transitions, one to move to `HEATING` and another to move to `COOLING`. 
    
        This model is simplified since the system always transitions through `STANDBY`, instead of transitioning directly from `HEATING` to `COOLING` or vice versa. 
        
    * The conditional transition logic takes into consideration the two temperature boundaries; one when the room is occupied and another when the room is unoccupied.

    * The following is an example where the model transitions from `HEATING`, which is labeled `stopHeating`. There are two Boolean expressions joined with a *OR* (`||`) logic. 

        In other words, this expression means "stop heating if the room is unoccupied and the room temperature is within the *unoccupied* threshold, OR if the room is occupied and the room is within the *occupied* threshold."

```js
(!$input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdOccupied)
```
## Validation steps
Before you complete this lesson, take a moment to validate that the solution works as intended:

1. Apply a temperature change to your device that will take it outside the comfort bounds.

    Make adjustments in the IoT Events detector model to see the LED strips on the side change to red (`HEATING`), blue (`COOLING`), or off (`STANDBY`). Use the warmth of your hands or a fan to raise or lower the detected temperature.
1. Alternatively, choose new event detector temperature thresholds that exclude your room's true ambient temperature. Afterward, redeploy the detector model to see it work. 
    Find the threshold values stored as variables in the model state labeled `STANDBY` under the `OnEnter` action labeled `setThresholds`. Update the numbers to anything that makes sense for your room and publish the changes to test. 

{{% notice info %}}
The temperature sensor is inside the housing of the kit and the code includes a hardcoded offset. You may notice a variance in the temperature reading up to 10 degrees between readings. 
{{% /notice %}}


**This completes the end-to-end solution!**


If the solution is working as expected, continue to the [**Conclusion**](/en/smart-thermostat/conclusion.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}