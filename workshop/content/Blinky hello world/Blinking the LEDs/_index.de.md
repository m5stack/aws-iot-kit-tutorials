+++
title = "Blinken der LEDs"
weight = 40
pre = "<b>d. </b>"
+++
Zu diesem Zeitpunkt läuft Ihre Gerätesoftware und Sie sind mit AWS IoT Core verbunden, senden Nachrichten und sind bereit, Nachrichten aus der Cloud zu empfangen und darauf zu reagieren. In diesem Kapitel abonnieren Sie ein [MQTT-Topic](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html), sehen die eingehenden Nachrichten in AWS IoT Core, und senden eine Nachricht, um die LEDs an Ihrem Gerät zu blinken. Da MQTT ein [Client-Server-Protokoll](https://mqtt.org/) ist, können Sie bestimmte Topics abonnieren und/oder Nachrichten an diese senden. Die zuvor im Registrierungsskript festgelegte Richtlinie schränkte die Themen ein, die das Gerät abonnieren und an die das Gerät senden kann. Dies ist wichtig für die Filterung und die Sicherheit. Ihr Gerät kann nur an Topics senden oder von den Topics empfangen, die mit der Client-ID übereinstimmen, was in diesem Fall die eindeutigen Seriennummer des sicheren Elements ist.

## Abonnieren über den AWS IoT MQTT-Client
Mit dem AWS IoT-MQTT-Client in der AWS IoT Core-Konsole können Sie MQTT-Nachrichten sowohl anzeigen als auch senden. Gehen Sie zunächst zu [AWS IoT-Konsole](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/) und wählen Sie **Test**, um die Clientansicht zu öffnen.

{{< img "aws_iot-mqtt_test_client-subscribe.en.png" "Choose test in AWS IoT console" >}}

Geben Sie im Feld **Themenfilter** `#` ein, um alle MQTT-Topics zu abonnieren. Der mehrstufige [Themenfilter Platzhalter](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#topicfilters) ist **#** und kann nur einmal als letztes Zeichen eines Themenfilters verwendet werden. Sobald Sie die Schaltfläche **Abonnieren** drücken, werden Nachrichten von Ihrem Gerät empfangen. Das Gerät darf nur Nachrichten zum Topic veröffentlichen, die mit `<<CLIENT_ID>>/` beginnen. Dies gibt einem anderen Abonnenten (z. B. Cloud-Anwendung) die Möglichkeit, Themen für bestimmte Clientgeräte zu filtern, und kann auch auf spezifischere Themen eingegrenzt werden (z. B. die Temperatur-Nachrichten eines spezifischen Geräts).

## Blinken der LED
Um das Blinken der LED-Leisten an den Seiten des M5Stack Core2 für AWS IoT EduKit zu starten/zu stoppen, werden wir vom AWS IoT MQTT-Client der Konsole eine Nachricht an ein Topic senden, das der Core2 abonniert hat. Das Topic für das Gerät hat das Muster `<<CLIENT_ID>>/#`. Sie können das Topic auf dem Gerät anzeigen, nachdem es das Topic erfolgreich abonniert hat. Darüber hinaus können Sie die Client-ID anzeigen, die auf dem seriellen Monitor des Host-Rechners ausgegeben wurde.

Geben Sie im Feld Veröffentlichen den folgenden Befehl ein, nachdem Sie den **<<CLIENT_ID>>** Text durch Ihre tatsächliche Client-ID ersetzt haben, die Sie gerade kopiert haben. Dann drücken Sie die Taste **Veröffentlichen**:
```
<<CLIENT_ID>>/blink
```
{{< img "aws_iot-mqtt_test_client-publish.en.png" "Subscribing to messages and publishing with AWS IoT console MQTT client" >}}
Auf Ihrem Gerät sollten jetzt die LEDs der Seitenleiste blinken. Um das Blinken zu unterbrechen, drücken Sie einfach erneut die Schaltfläche **Veröffentlichen**, um zum selben Topic zu veröffentlichen.

{{% notice info %}}
Um den seriellen Monitor zu verlassen, drücken Sie **STRG** + **C**.
{{% /notice %}}

## Aufräumen
Um die Ressourcennutzung zu optimieren und unerwünschte AWS-Cloud-Servicegebühren zu vermeiden, löschen Sie den Flash auf Ihrem Gerät, um es für die nachfolgenden Tutorials vorzubereiten. Um den Flash zu löschen, müssen Sie sich in einem Projekt befinden, das bereits erstellt wurde (z. B. dem, das Sie gerade abgeschlossen haben), den laufenden seriellen Monitor beenden, der den Port möglicherweise blockiert (**STRG** + **C** drücken) und den Befehl verwenden:

```
pio run --environment core2foraws --target erase
```


Weiter zum letzten Kapitel, dem [**Abschluss**](conclusion.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}