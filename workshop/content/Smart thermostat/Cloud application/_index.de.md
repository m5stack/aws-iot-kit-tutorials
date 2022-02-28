+++
title = "Cloud-Anwendung"
weight = 50
pre = "<b>e. </b>"
+++

## Einführung in das Kapitel
Am Ende dieses Kapitels kann Ihre Lösung Folgendes:

* Telemetrie-Nachrichten vom Core2 des AWS IoT EduKit-Gerät in einer Cloud-Anwendung verarbeiten.
* Sie bestimmen, in welchem Zustand sich das HLK-System befinden soll: HEATING, COOLING oder STANDBY.
* Senden Sie eine gewünschte Statusmeldung basierend auf verarbeiteten Eingaben an Ihren Geräteschatten (Shadow).
* Synchronisieren Sie den gewünschten Shadow-Status mit Ihrem Gerät.
* Aktualisieren Sie die Geräteanzeige mit dem gewünschten HLK-Systemzustand.

## So erstellen Sie die Cloud-Anwendung
In diesem Kapitel erstellen Sie eine serverlose Anwendung ohne Code, die die Eingaben Ihres intelligenten Thermostats analysiert und das Verhalten für das entsprechende (fiktive) HLK-System bestimmt. Sie verwenden IoT-Events, um eine Ressource bereitzustellen, die als Detektor-Modell bezeichnet wird, und die weitergeleiteten Nachrichten verarbeitet. Bewerten Sie, ob Zustandsänderungen von Heizung über Kühlung bis Standby auftreten sollten, und senden dann eine Nachricht mit der aktualisierten Statusänderung an Ihr intelligentes Thermostat zurück, falls vorhanden.

Hier ist eine Vorschau des Detektor-Modells, das Sie erstellen werden:
{{< img "iot-events-detector_model.en.png" "Detector model" >}}

Wie Sie sehen, gibt es drei Zustände für die HLK-Anwendung: HEATING, COOLING und STANDBY. Die Applikation befindet sich immer in einem dieser Status und beginnt in *standby*. Wenn neue Nachrichten vom intelligenten Thermostat eingehen, wertet das Detektor-Modell diese Eingaben anhand der bedingten Logik aus, um zu bestimmen, ob das Modell in einen neuen Zustand übergehen soll.

Wenn beispielsweise im Standby-Zustand ein neuer Temperaturwert über 80 (in Grad Fahrenheit) liegt, bewertet das Modell einen Übergang in den Zustand *cooling*. Wenn es dann in den Status *cooling* wechselt, veröffentlicht das Modell eine neue Nachricht zurück an IoT Core, um den Geräteschatten des intelligenten Thermostats mit einer Meldung wie: `{"state": {"desired": {"hvacStatus": "COOLING" }}}` zu aktualisieren. Basierend auf der Arbeit, die Sie ursprünglich im Kapitel **Datensynchronisierung** geleistet haben, werden diese neuen Befehle bestätigt, der Status auf dem Gerät entsprechend aktualisiert und die LED-Streifen auf Rot, Blau oder Aus gesetzt.

Es sprengt den Rahmen dieses Lernmoduls, vollständig zu untersuchen, wie Detektor-Modelle in IoT-Events erstellt werden. Stattdessen bringen die folgenden Schritte Ihr Detektor-Modell zum Laufen. Ein paar Anmerkungen zu den wichtigsten Teilen sollen Ihnen helfen zu verstehen, wie _dieses_ Modell funktioniert. Eine Einführung in die ersten Schritte mit AWS IoT Events finden Sie im [Entwicklerhandbuch](https://docs.aws.amazon.com/iotevents/latest/developerguide/what-is-iotevents.html).

Führen Sie die folgenden Schritte aus, um das in der Abbildung oben dargestellte Detektor-Modell zu erstellen:

Erstellen Sie zunächst auf Ihrem lokalen Desktop eine Datei mit folgendem Inhalt. Nennen Sie die Datei etwas wie `model.json`. Sie müssen die Datei bearbeiten und die drei Instanzen des Begriffs **<<CLIENT_ID>>** durch die Client-ID/Seriennummer Ihres Geräts ersetzen (sie wird auf dem Bildschirm Ihres Geräts angezeigt).

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
Verwenden Sie als Nächstes diese Datei, um ein neues Detektor-Modell in die AWS IoT Events Konsole zu importieren:
1. Gehen Sie zur AWS IoT-Events-Konsole.
2. Wählen Sie **Detektor-Modelle**.
3. Wählen Sie **Aktion** und dann **Detektor-Modell importieren**.
4. Wählen Sie **Importieren**.
5. Suchen Sie die Datei, die Sie im vorherigen Schritt erstellt haben, und wählen Sie **Öffnen** (der Name der Schaltfläche kann je nach Browser variieren).
6. Wählen Sie **Veröffentlichen**.
7. Geben Sie unter *Rolle* einen neuen Namen an, der zum Erstellen einer neuen IAM-Rolle verwendet wird, wie `edukit-iotevents`. Die Konsole erstellt eine neue IAM-Rolle, die IoT-Events die Berechtigung zum Betrieb Ihres Modells erteilt.
8. Wählen Sie **Speichern und veröffentlichen**.

Sie haben das Detektor-Modell jetzt in IoT-Events erstellt. Wenn neue Nachrichten von Ihrem intelligenten Thermostat veröffentlicht werden, leitet die IoT-Core-Regel, die Sie im vorherigen Kapitel erstellt haben, sie an die IoT-Events Eingabe weiter. Anschließend überträgt IoT Events Kopien der Nachrichten von allen Eingaben an alle Detektor-Modelle.

Nun zu der Erklärung einiger wichtiger Teile des Modells. Jeder Zustand des Modells (Heizen/Kühlen/Standby) ist nahezu gleich. Der Status *standby* hat eine zusätzliche Aktion zum Festlegen der numerischen Schwellenwerte, die jeder Zustand verwendet, um zu bestimmen, ob er den Status ändern soll. Diese könnten einmal in einem einmaligen Initialisierungszustand gesetzt werden, bevor sie zum ersten Mal in *Standby* verändert werden. Hier sind sie in *Standby* enthalten, um die Demonstration zu vereinfachen.

Die Konfiguration der übrigen Zustände ist ähnlich. Es gibt ein *OnEnter*-Ereignis, um eine Meldung im Geräteschatten des intelligenten Thermostats zu veröffentlichen, die angibt, in welchem Zustand sich die HLK jetzt befinden soll. Es gibt einen bedingten Übergang, der nach der Verarbeitung jeder Nachricht ausgewertet wird und prüft, ob sich der Status ändern soll. Wenn sich das Modell beispielsweise im Zustand *heating* befindet und der bedingte Ausdruck für *StopHeating* den Wert true ergibt, wechselt das Modell in den Status *Standby*. Beachten Sie, dass der Status *Standby* zwei Übergänge hat, jeweils einen für den Übergang zu*Heizen* oder *Kühlung*. Dieses Modell ist insofern vereinfacht, als das System immer den Status *Standby* durchläuft, anstatt direkt von *Heizen* zu *Kühlung* oder umgekehrt. Die bedingte Übergangslogik berücksichtigt die beiden Temperaturgrenzen; eine, wenn der Raum belegt ist, und eine, wenn der Raum nicht belegt ist.

Im Folgenden finden Sie ein Beispiel für einen Übergang vom Zustand *heating* mit der Bezeichnung *stopHeating*. Es gibt zwei boolesche Ausdrücke, die mit der ODER `||` -Logik verknüpft sind. Im Klartext bedeutet dieser Ausdruck „Stoppen Sie die Heizung, wenn der Raum nicht belegt ist und die Raumtemperatur warm genug ist, wenn Sie nicht besetzt sind, ODER wenn der Raum belegt ist und der Raum warm genug für Menschen ist“.

```js
(!$input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdOccupied)
```

Wenn alles gemäß den Anweisungen dieses Moduls konfiguriert wurde, sollten jetzt Statusänderungen in Form von aktualisierten LED-Streifen an Ihren intelligenten Thermostat gesendet werden. Damit ist die End-to-End-Lösung abgeschlossen!

## Schritte zur Validierung
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob die Lösung wie beabsichtigt konfiguriert ist:

1. Wenden Sie eine Temperaturänderung auf Ihr Gerät an, die es außerhalb der im IoT-Events Detektor-Modells festgelegten Komfortgrenzen bringt, um zu sehen, wie die LED-Streifen an der Seite in Rot (Heizung), Blau (Kühlung) oder Aus (Standby) wechseln. Verwenden Sie die Wärme Ihrer Hände oder die Kühlung eines Ventilators, um die gemessene Temperatur zu erhöhen oder zu senken.
1. Wählen Sie alternativ neue Temperaturschwellenwerte für Ihren Ereignisdetektor, die die tatsächliche Umgebungstemperatur Ihres Raums ausschließen, und stellen Sie Ihr Detektor-Modell erneut ein, damit es funktioniert. Sie finden die als Variablen gespeicherten Schwellenwerte im Modellzustand mit der Bezeichnung „Standby“ unter der *OnEnter*-Aktion mit der Bezeichnung „setThresholds“. Sie können diese Zahlen beliebig aktualisieren und die Änderungen veröffentlichen.

Hinweis: Der Temperatursensor befindet sich im Gehäuse des Kits und der Code enthält einen fest codierten Offset. Sie können eine Abweichung der Temperaturmessung von bis zu 10 Grad vom Ablesen bis zum Messwert feststellen.

Wenn die Lösung wie erwartet funktioniert, können Sie mit der [**Zusammenfassung**](/de/smart-thermostat/conclusion.html) fortfahren.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}