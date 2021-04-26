+++
title = "Cloud-Anwendung"
weight = 50
pre = "<b>e. </b>"
+++

## Kapiteleinleitung
Am Ende dieses Kapitels wird Ihre Lösung Folgendes können:

* Verarbeiten von Telemetrie-Nachrichten vom "Core2 for AWS IoT EduKit"-Gerät in einer verwalteten Cloud-Anwendung
* Bestimmen in welchem Zustand sich das HLK-System befinden soll: HEIZEN, KÜHLEN, oder STANDBY
* Senden einer gewünschten Zustandsmeldung an Ihren Geräteschatten basierend auf verarbeiteten Eingängen
* Synchronisieren des gewünschten Schattenzustandes mit Ihrem Gerät
* Aktualisieren des Gerätedisplays mit dem gewünschten HLK-Systemzustand

## Wie Sie die Cloud-Anwendung erstellen
In diesem Kapitel werden Sie eine serverlose Anwendung ohne jeglichen Code erstellen, die die Eingaben von Ihrem intelligenten Thermostat analysiert und das Verhalten für das entsprechende (fiktive) HLK-System bestimmt. Sie werden IoT-Events verwenden, um eine Ressource namens Detektormodell bereitzustellen, die die von Ihrer IoT-Core-Regel weitergeleiteten Geräteschattennachrichten verarbeitet, auswertet, ob eine Zustandsänderung von Heizen zu Kühlen zu Standby erfolgen sollte, und dann eine Nachricht mit der aktualisierten Zustandsänderung, falls vorhanden, an Ihren intelligenten Thermostat zurücksendet.

Hier sehen Sie eine Vorschau auf das zu erstellende Detektormodell:
{{< img "iot-events-detector_model.en.png" "Detector model" >}}

Wie Sie sehen können, gibt es drei Zustände für die HLK-Anwendung: Heizen, Kühlen und Standby. Die Anwendung befindet sich immer in einem dieser Zustände und wird mit dem Standby-Zustand initialisiert. Wenn neue Meldungen vom intelligenten Thermostat eintreffen, wertet das Detektormodell die Eingaben anhand der bedingten Logik aus, um zu bestimmen, ob das Modell in einen neuen Zustand übergehen soll.

Wenn zum Beispiel aus dem *Standby*-Zustand eine neue Temperaturmessung über 80 (in Grad Fahrenheit) eintrifft, bewertet das Modell einen Übergang in den *cooling* Zustand. Beim Eintritt in den *cooling* Zustand veröffentlicht das Modell dann eine neue Nachricht zurück an IoT Core, um den Geräteschatten des intelligenten Thermostats mit einer Nachricht wie folgt zu aktualisieren: `{ "state": { "desired": { "hvacStatus": "COOLING" } } }`. Basierend auf der Arbeit, die Sie anfangs im Kapitel **Datensynchronisation** durchgeführt haben, werden diese neuen Befehle auf dem gewünschten Knoten bestätigt und der Status auf dem Gerät aktualisiert, um den Befehl wiederzugeben, indem die LED-Streifen auf rot, blau oder aus aktualisiert werden.

Es würde den Rahmen dieses Lernmoduls sprengen, die Erstellung von Detektormodellen in IoT-Events vollständig zu erläutern. Stattdessen finden Sie im Folgenden Schritte, um Ihr Detektormodell einzurichten und zum Laufen zu bringen, mit einigen Anmerkungen zu wichtigen Teilen, damit Sie wissen, wie _dieses_ Modell funktioniert. Eine Einführung in die ersten Schritte mit AWS IoT Events finden Sie im [Entwicklerhandbuch](https://docs.aws.amazon.com/iotevents/latest/developerguide/what-is-iotevents.html).

Führen Sie die folgenden Schritte aus, um das in der obigen Abbildung gezeigte Detektormodell zu erstellen.

Zunächst erstellen Sie auf Ihrem lokalen Desktop eine Datei mit folgendem Inhalt. Nennen Sie die Datei z. B. `model.json`. Sie müssen die Datei bearbeiten, um die drei Instanzen des Begriffs **<<CLIENT_ID>>** zu finden und durch die Client-ID/Seriennummer Ihres Geräts zu ersetzen (sie ist auf dem Bildschirm Ihres Geräts aufgedruckt).

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

Als Nächstes werden Sie diese Datei in den Schritten zum Importieren eines neuen Detektormodells in der AWS IoT Events-Konsole verwenden.

1. Gehen Sie zur Management-Konsole von AWS IoT Events.
2. Wählen Sie **Detector models**.
3. Wählen Sie **Action**  und dann **Import detector model**.
4. Wählen Sie **Import**.
5. Suchen Sie die Datei, die Sie im vorherigen Schritt erstellt haben, und wählen Sie **Open** (der Name der Schaltfläche kann je nach Browser variieren).
6. Wählen Sie **Publish**.
7. Geben Sie unter *Role* einen neuen Namen an, der zum Erstellen einer neuen IAM-Rolle verwendet wird, z. B. `edukit-iotevents`. Die Konsole erstellt eine neue IAM-Rolle, die IoT Events die Berechtigung zum Betrieb Ihres Modells gibt.
8. Wählen Sie **Save and publish**.

Sie haben nun das Detektormodell für IoT Events erstellt. Wenn neue Nachrichten von Ihrem intelligenten Thermostat veröffentlicht werden, leitet die IoT Core-Regel, die Sie im vorherigen Kapitel erstellt haben, diese an die IoT Events-Eingangsressource weiter, dann pusht IoT Events Kopien der Nachrichten von allen Eingängen an alle konsumierenden Detektormodelle, wie dieses, zur Auswertung aller Zustandsänderungen.

Nun zur Erklärung einiger wichtiger Teile des Modells. Jeder Zustand des Modells (Heizen/Kühlen/Standby) ist nahezu identisch. Der *standby*-Zustand hat eine zusätzliche Aktion, um die numerischen Schwellenwerte zu setzen, die jeder Zustand verwendet, um zu bestimmen, ob er den Zustand wechseln soll. Diese könnten einmal in einem einmaligen Initialisierungszustand gesetzt werden, bevor sie zum ersten Mal in den *standby*-Zustand wechseln, aber sie sind der Einfachheit halber im *standby*-Zustand enthalten.

Der Rest der Konfiguration der Zustände ist ähnlich. Es gibt ein OnEnter-Ereignis, um eine Nachricht an den Geräteschatten des Smart-Thermostats zu veröffentlichen, die angibt, in welchem Zustand sich die HLK jetzt befinden soll. Es gibt einen bedingten Übergang, der nach der Verarbeitung jeder Nachricht ausgewertet wird und prüft, ob der Zustand geändert werden soll. Wenn sich das Modell z. B. im *heating*-Zustand befindet und der bedingte Ausdruck für *stopHeating* als wahr ausgewertet wird, wechselt das Modell in den *standby*-Zustand. Beachten Sie, dass der *standby*-Zustand zwei Übergänge hat, jeweils einen für den Übergang zum *heating*-Zustand oder *cooling*-Zustand. Dieses Modell ist insofern vereinfacht, als das System immer den *standby*-Zustand durchläuft, anstatt direkt vom Heizen zum Kühlen oder umgekehrt zu wechseln. Die bedingte Übergangslogik berücksichtigt die zwei Temperaturgrenzen; eine, wenn der Raum belegt ist und eine weitere, wenn der Raum nicht belegt ist.

Nachfolgend ist ein Beispiel für einen Übergang vom Zustand *heating* mit der Bezeichnung *stopHeating* dargestellt. Es gibt zwei boolesche Ausdrücke, die mit ODER `||` Logik verbunden sind. Im Klartext bedeutet dieser Ausdruck: "Stoppe die Heizung, wenn der Raum unbesetzt ist und die Raumtemperatur warm genug ist, während er unbesetzt ist, ODER wenn der Raum besetzt ist und der Raum warm genug für Personen ist."

```js
(!$input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdUnoccupied) || ($input.thermostat.current.state.reported.roomOccupancy && $input.thermostat.current.state.reported.temperature > $variable.heatingThresholdOccupied)
```

Wenn alles gemäß den Anweisungen dieses Moduls konfiguriert wurde, sollten Sie jetzt Statusänderungen sehen, die in Form von aktualisierten LED-Streifen an Ihr intelligentes Thermostat geliefert werden, wodurch die End-to-End-Lösung abgeschlossen ist!

## Validierungsschritte
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie folgendermaßen überprüfen, ob die Lösung wie vorgesehen konfiguriert ist:

1. Wenden Sie eine Temperaturänderung auf Ihr Gerät an, die es außerhalb der im IoT-Ereignismelder-Modell eingestellten Komfortgrenzen bringt, um zu sehen, dass die LED-Streifen an der Seite auf rot (Heizung), blau (Kühlung) oder aus (für Standby) wechseln. Verwenden Sie die Wärme Ihrer Hände oder einen Ventilator, um die erkannte Temperatur zu erhöhen oder zu senken.
2. Wählen Sie alternativ neue Temperaturschwellenwerte für Ihren Ereignismelder, die die tatsächliche Umgebungstemperatur Ihres Raums ausschließen, und setzen Sie Ihr Meldermodell erneut ein, um zu sehen, ob es funktioniert. Sie finden die Schwellenwerte als Variablen im Modellzustand mit der Bezeichnung "standby" unter der OnEnter-Aktion mit der Bezeichnung "setThresholds" gespeichert. Sie können diese Zahlen auf jeden Wert aktualisieren, der für Ihren Raum sinnvoll ist, und die Änderungen zum Testen veröffentlichen.

Hinweis: Der Temperatursensor befindet sich im Inneren des Gehäuses des Bausatzes und der Code enthält einen hartkodierten Offset. Die Temperaturanzeige kann von Messung zu Messung um bis zu 10 Grad variieren.
Wenn die Lösung wie erwartet funktioniert, können Sie mit der [**Zusammenfassung**](/de/smart-thermostat/conclusion.html) fortfahren.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}