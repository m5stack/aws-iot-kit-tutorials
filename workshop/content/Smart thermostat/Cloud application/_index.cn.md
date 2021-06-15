+++
title = "云应用程序"
weight = 50
pre = "<b>e. </b>"
+++

## 章节简介
学完本章节后，您的解决方案将能够：

* 在托管的云应用程序中处理来自 Core2 for AWS IoT EduKit 设备的遥测数据。
* 确定 HVAC 系统应设置为何种状态：制热、制冷或待机。
* 根据处理的输入数据向设备影子发送期望的状态消息。
* 将期望的影子状态同步到设备。
* 使用期望的 HVAC 系统状态更新设备显示内容。

## 使用所需的 HVAC 系统状态更新设备显示内容
在这一章节，您将构建无服务器应用程序，分析来自智能恒温器的输入数据并确定相应（虚构）的 HVAC 系统的行为。您将使用 IoT Events 来部署检测器模型这一资源来处理 IoT Core 规则转发的设备影子消息，评估是否需要将状态从制热切换到制冷再切换到待机，然后将最新的状态更改（如果有）以消息的形式发送回智能恒温器。

我们先来了解一下要创建的检测器模型：
{{< img "iot-events-detector_model.en.png" "Detector model" >}}

如您所见，HVAC 应用程序有三种状态：制热、制冷和待机。应用程序会处于其中一种状态并且初始化为 *standby（待机）* 状态。来自智能恒温器的新消息送达后，检测器模型会对照条件逻辑来评估输入数据，以便确定模型是否应该转换到新的状态。

例如，之前处于 *待机* 状态，但如果新的温度读数高于 80 华氏度，那么模型评估结果为需要转换到 *制冷* 状态。进入 *制冷* 状态后，模型会向 IoT Core 发送一条新消息，来更新智能恒温器的设备影子，消息内容类似于：`{ "state": { "desired": { "hvacStatus": "COOLING" } } }`。根据您最初在 **数据同步** 章节所做的工作，新命令会得到确认，并且设备上的状态也会据此进行更新，方式是 LED 灯变为红色、蓝色或关闭，以便体现出命令执行结果。

本学习模块不详细讲解如何在 IoT Events 中创建检测器模型。但是，您可以参照以下步骤来启动和运行检测器模型，您还可以了解到一些有关主要内容的说明，以便了解 _该模型_ 的工作原理。如果您刚刚开始使用 AWS IoT Events，请参阅 [开发人员指南](https://docs.aws.amazon.com/iotevents/latest/developerguide/what-is-iotevents.html)。

要创建上图中显示的检测器模型，请完成以下步骤：

首先，在本地桌面上创建一个文件，文件内容如下。文件名可以设置为 `model.json`，您务必使用您的设备的客户端 Id/序列号（显示在您的设备屏幕上）替换 **<<CLIENT_ID>>**。

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

接下来，使用该文件在 AWS IoT Events 控制台中导入新的检测器模型：
1. 登录 AWS IoT Events 管理控制台。
2. 选择 **Detector models（检测器模型）**。
3. 选择 **Action（操作）**，然后选择 **Import detector model（导入检测器模型）**。
4. 选择 **Import（导入）**。
5. 找到您在上一步中创建的文件，然后选择 **Open（打开）**（按钮名称因浏览器而异）。
6. 选择 **Publish（发布）**。
7. 在 *Role（角色）* 下，提供用于创建新的 IAM 角色的新名称，例如 `edukit-iotevents`。控制台将创建新的 IAM 角色，该角色授予 IoT Events 操作模型的权限。
8. 选择 **Save and publish（保存并发布）**。

您已经将检测器模型部署到 IoT Events。智能恒温器发布新消息后，您在上一章节中创建的 IoT Core 规则会将消息转发给 IoT Events 输入资源，然后 IoT Events 会将来自任何输入的消息副本推送到任何使用数据的检测器模型，就像当前这个一样，用于评估是否需要更改状态。

我们来了解一下有关模型的几项重要内容。模型的每种状态（制热/制冷/待机）都几乎相同。*待机* 状态有一项额外的操作，即设置数值阈值，用于确定是否需要更改状态。您可以在第一次进入 *待机* 状态之前，在初始化状态下设置一次数值阈值，但是，为了便于演示，*待机* 状态中已经包含了这些值。

其他状态配置也类似。OnEnter 事件会向智能恒温器的设备影子发布消息，说明 HVAC 系统应进入何种状态。系统处理每一条消息后，会评估条件转换，以便确定是否需要更改状态。例如，如果模型处于 *制热* 状态，并且针对 *stopHeating* 的条件表达式评估为真，则模型将转换为 *待机* 状态。请注意，*待机* 状态涉及两种转换，一种是转换为 *制热*，另一种是转换为 *制冷*。我们对模型进行了简化，因为系统总是要从 *待机* 状态转换到 *制热* 状态或 *制冷* 状态，而不是直接从 *制热* 状态转换到 *制冷* 状态或直接从 *制冷* 状态转换到 *制热* 状态。条件转换逻辑考虑到了两个温度界限：空间内有人时的温度界限和空间内无人时的温度界限。

以下是从 *制热* 状态（标记为 *stopHeating*）进行转换的示例。有两个与 OR `||` 逻辑关联的布尔表达式。简而言之，表达式的意思是 “如果空间内没有人，并且空间内没有人时也足够暖和，则停止制热，或者如果空间内有人，并且足够暖和，则停止制热。”

```js
(!$input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdOccupied)
```

如果您已经按照本模块中的说明完成了所有配置，那么现在应该能够看到发送到智能恒温器的状态更改信息，其形式为更新 LED 灯条状态，至此，我们已经完成了端到端的解决方案！

## 验证步骤
在进入下一章节之前，您可以验证您的解决方案是否已按预期配置完毕，方法是：

1. 改变设备温度，使其超出 IoT Events 检测器模型中设置的舒适温度界限，然后可以看到 LED 灯条变为红色（制热）、蓝色（制冷）或关闭（待机）。借助手或风扇来提高或降低检测到的温度。
1. 或者，为事件检测器选择新的温度阈值，不考虑空间的真实环境温度并且重新部署检测器模型，观察它如何运作。您可以在标有 “setThresholds” 的 OnEnter 操作下找到作为变量存储在 “待机” 模型状态下的阈值。 您可以将数字更新为任何符合空间情况的内容并将更改发布到测试环境中。

请注意：温度传感器位于套件外壳内，代码包含硬编码的偏移。不同的温度读数之间的偏差可能高达 10 度。

如果解决方案符合预期，就可以进入 [**总结**](/cn/smart-thermostat/conclusion.html) 部分。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}