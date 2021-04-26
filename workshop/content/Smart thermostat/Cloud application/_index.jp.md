+++
title = "クラウドアプリケーション"
weight = 50
pre = "<b>e. </b>"
+++

## 本章の概要
この章の終わりには、ソリューションで次のようなことができるようになります。

* マネージドクラウドアプリケーションの Core2 for AWS IoT EduKit デバイスからのテレメトリメッセージを処理する
* HVAC システムの状態 (HEATING、COOLING、または STANDBY) を特定する
* 処理された入力に基づき、Device Shadow に希望する状態のメッセージを送信する
* シャドウ状態をデバイスと同期する
* 指示された HVAC システム状態を使用してデバイスディスプレイを更新する

## クラウドアプリケーションの構築方法
この章では、スマートサーモスタットからの入力を分析したり、対応するHVAC システムの動作を特定したりするコードを作成せずに、サーバーレスアプリケーションを構築します。IoT Events を使用して探知器モデルと呼ばれるリソースをデプロイします。このリソースは、IoT Core ルールによって転送された Device Shadow メッセージを処理し、状態が「heating」、「cooling」、または「standby」に変更された場合に評価して、状態変更の更新についてスマートサーモスタットにメッセージを返信します。

以下は作成する探知器モデルです

{{< img "iot-events-detector_model.en.png" "Detector model" >}}

ご覧のとおり、HVAC アプリケーションには「heating」、「cooling」、「standby」の 3 つの状態が存在します。アプリケーションは常にいずれかの状態にあり、「standby」状態に初期化されます。スマートサーモスタットから新しいメッセージが到着すると、探知器モデルは条件付きロジックに対して入力を評価し、モデルが新しい状態に移行すべきかどうかを決定します

例えば、「standby」状態であっても、華氏 80 度 (摂氏 26 度) 以上の新しい温度の読み込みが到着した場合、モデルは「cooling」状態への移行を評価します。「cooling」状態に入ると、モデルは IoT Core に新しいメッセージをパブリッシュし、`{ "state": { "desired": { "hvacStatus": "COOLING" } } }` のようなメッセージを使用して、スマートサーモスタットの Device Shadow の更新を求めます。「データ同期」の章で行った作業に基づき、希望するノードでのこれらの新しいコマンドは受信され、デバイスの状態は、LEDを赤、青、またはオフに更新することで、コマンドを反映するよう更新されます。

IoT Events での探知器モデル作成方法の詳細は、このワークショップの範疇外です。その代わり、以下に探知器モデルを開始、実行するためのステップを示します。このモデルの動作方法を理解できるよう、主要な箇所にはヒントが記載されています。AWS IoT Events の利用を開始するには、[デベロッパーガイド](https://docs.aws.amazon.com/iotevents/latest/developerguide/what-is-iotevents.html)や、[AWS IoT Events初級ハンズオン](https://aws-iot-events-for-beginners.workshop.aws/)をご確認ください。

上の画像に表示された探知器モデルを作成するには、次のステップを完了してください。

まず、次のコンテンツを使用して、ローカルパソコンにファイルを作成します。ファイルに `model.json` などの名前を付けます。このJSONドキュメントに**<<CLIENT_ID>>**が複数ありますが、この部分はデバイス画面やシリアルモニター出力に表示された Core2 for AWS のシリアル番号に書き換えます。

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

次に、AWS IoT Events コンソールに新しい探知器モデルをインポートするステップで、このファイルを使用します。
1. [IoT Events マネジメントコンソール](https://us-west-2.console.aws.amazon.com/iotevents/home?region=us-west-2) に移動します。
2. [Detector models (探知器モデル)] を選択します。
3. [Action (アクション)] を選択し、[Import detector model (探知器モデルのインポート)] を選択します。
4. [Import] (インポート) を選択します。
5. 前回のステップで作成したファイルを見つけ、[Open (開く)] を選択します (ボタンの名前はブラウザによって異なる可能性があります)。
6. [Publish (発行)] を選択します。
7. [Role (ロール)] で、新しい IAM ロールの作成に使用される新しい名前を入力します (edukit-iotevents など)。コンソールが新しい IAM ロールを作成します。これにより、IoT Events にモデル操作の許可が付与されます。
8. [Save and publish (保存して発行)] を選択します。

これで IoT Events に探知器モデルがデプロイされました。新しいメッセージがスマートサーモスタットからパブリッシュされると、前の章で作成した IoT Core ルールがメッセージを IoT Events 入力リソースに転送します。その後 IoT Events がメッセージのコピーを、入力からこのような利用中の探知器モデルにプッシュし、状態の変更を評価します。

モデルの主要な特徴について説明します。モデルの各状態 (heating、cooling、standby) はほとんど同じです。**standby** 状態には、状態を変更すべきかどうか決定するために、すべての状態が使用する数のしきい値を設定するという、追加のアクションが 1 つ存在します。これらは、初めて **standby** に移行する前に、1 度の初期化状態において設定することもできますが、デモを簡素化するため **standby** に含まれています。

残りの状態の設定は似通っています。メッセージをスマートサーモスタットの Device Shadow にパブリッシュする OnEnter イベントがあります。これは HVAC の現在の状態を示します。各メッセージが処理された後評価される、条件付きの移行もあります。これは状態を変更すべきかどうかを確認します。例えば、モデルが **heating** 状態にあり、**stopHeating** の条件付きの式が **true** と評価した場合、モデルは **standby** 状態に移行します。**standby** 状態には 2 つの移行があり、それぞれ **heating** または **cooling** に移行します。このモデルでは、直接 heating から cooling またはその反対に移動するのではなく、常に **standby** 状態を通過するようシステムで簡素化されています。条件付きの移行ロジックは、2 つの温度の境界を考慮します。1 つは部屋に人がいるとき、もう 1 つは部屋に人がいないときです。

以下は **stopHeating** とラベル付けされた **heating** 状態からの移行のサンプルです。2 つのブーリアン式が OR `||` ロジックで結びつけられています。この式は、「部屋に人がおらず、人がいない期間の部屋の温度がある程度暖かい場合、または部屋に人がおり、人にとって部屋が十分に暖かい場合、暖房を停止する」ことを表しています。

```js
(!$input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdOccupied)
```

すべてがこのモジュールの手順どおり設定されている場合、LEDが更新されることでスマートサーモスタットの状態の変更を確認でき、エンドツーエンドのソリューションが完成します。

## 検証ステップ
次の章に進む前に、ソリューションが想定どおり設定されているかを検証できます

1. IoT Events 探知器モデルで設定されている適正温度以上または以下となる温度変更をデバイスに適用し、側面の LEDが赤 (heating) または青 (cooling) に変わるか、オフ (standby) になるかを確認します。手の温もりまたはファンを使用して、検知される温度を高くしたり低くしたりしてみます
1. または、イベント検知で、環境温度を除外した新しい温度しきい値を選択し、探知器モデルを再デプロイして問題なく動作しているかどうかを確認します。しきい値は変数としてモデルの状態に保存され、「setThresholds」とラベル付けされた OnEnter アクションで「standby」とラベル付けされていることが確認できます。 これらの数字は、部屋に対して適切な温度に更新することができ、変更を公開してテストすることも可能です。

注: 温度センサーはキットのハウジング内に存在し、コードにはハードコードのオフセットが含まれます。読み込みによって、温度には華氏 10 度 (摂氏 5 度) ほどの差が生じる可能性があります。

ソリューションが予測どおりに動作している場合、[まとめ](/jp/smart-thermostat/conclusion.html)に移動します。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}