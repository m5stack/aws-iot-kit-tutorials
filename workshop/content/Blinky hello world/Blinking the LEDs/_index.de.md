+++
title = "Blinken der LEDs"
weight = 40
pre = "<b>d. </b>"
+++

Zu diesem Zeitpunkt läuft Ihre Gerätesoftware und Sie sind mit AWS IoT Core verbunden, senden Nachrichten und sind bereit, Nachrichten aus der Cloud zu empfangen und darauf zu reagieren. In diesem Kapitel werden Sie ein MQTT-Thema abonnieren, die in AWS IoT Core eingehenden Nachrichten anzeigen und eine Nachricht zum Blinken Ihrer Geräte-LEDs senden. Da MQTT ein Publish-Subscribe-Protokoll ist, können Sie bestimmte Themen abonnieren und/oder veröffentlichen. Die Richtlinie, die zuvor im Registrierungsskript festgelegt wurde, schränkt die Themen ein, die das Gerät abonnieren und veröffentlichen kann. Dies ist wichtig für die Filterung und möglicherweise für die Sicherheit. Ihr Gerät kann nur an Themen senden oder empfangen, die mit der Client-ID übereinstimmen, die in diesem Fall mit der eindeutigen Seriennummer des sicheren Elements des Geräts identisch ist.

## Abonnieren über den AWS IoT MQTT-Client
Mit dem AWS IoT MQTT-Client in der AWS IoT Core-Konsole können Sie MQTT-Nachrichten sowohl anzeigen als auch veröffentlichen. Um zu beginnen, gehen Sie zur AWS [IoT-Konsole](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/) und wählen Sie **Test**, um die Client-Ansicht zu öffnen.

{{< img "aws_iot-mqtt_test_client-subscribe.en.png" "AWS IoT MQTT test client" >}}

Geben Sie im Feld **Subscription topic** `#` ein, um alle MQTT-Topic-Namen zu abonnieren. Der mehrstufige [Wildcard-Topic-Filter](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#topicfilters) ist **#** und kann nur einmal als letztes Zeichen eines Topic-Filters verwendet werden. Sobald Sie auf die Schaltfläche **Subscribe to topic** klicken, sehen Sie, dass Nachrichten von Ihrem Gerät gesendet werden. Das Gerät darf nur Nachrichten zu dem Thema veröffentlichen, das mit `<<CLIENT_ID>>/` beginnt. Dies gibt einem anderen Teilnehmer (z. B. einer Cloud-Anwendung) die Möglichkeit, Themen für bestimmte Client-Geräte zu filtern, und kann auch auf spezifischere Themen eingegrenzt werden (z. B. Temperaturmessung eines bestimmten Geräts).

## Blinken der LED
Um das Blinken der LED-Balken an den Seiten der M5Stack Core2 for AWS IoT EduKit-Referenzhardware zu starten/stoppen, werden wir vom AWS IoT MQTT-Client der Konsole aus ein Thema veröffentlichen, das der Core2 for AWS IoT EduKit abonniert hat. Dazu müssen wir zunächst die Client-ID des Geräts abrufen. Gehen Sie zurück zur Shell-Eingabeaufforderung, wo der serielle Monitor noch läuft, und kopieren Sie die **Client-ID**.

{{% notice note %}}
Wenn Sie an Ihrer Shell-Eingabeaufforderung kein **(edukit)** Präfix sehen, stellen Sie sicher, dass Sie Ihre conda-Umgebung aktivieren, indem Sie `conda activate edukit` ausführen. Wenn der Befehl idf.py nicht gefunden wird, fügen Sie das ESP-IDF mit dem Befehl `. $HOME/esp/esp-idf/export.sh` (macOS/Linux) oder `%userprofile%\Desktop\esp-idf\export.bat` (Windows) zu Ihrem Pfad hinzu.
{{% /notice %}}

Geben Sie im Feld "Veröffentlichen" den unten stehenden Befehl ein, ersetzen Sie jedoch den Text **<<CLIENT_ID>>** durch Ihre tatsächliche Client-ID, die gerade kopiert wurde, und drücken Sie dann die Schaltfläche **Publish to topic**:
```
<<CLIENT_ID>>/blink
```
{{< img "aws_iot-mqtt_test_client-publish.en.png" "Subscribing to messages and publishing with AWS IoT console MQTT client" >}}
Auf Ihrem Gerät sollten nun die LEDs der Seitenleiste blinken. Um das Blinken zu unterbrechen, drücken Sie einfach erneut die Schaltfläche **Publish to topic** zum gleichen Thema und es wird die blinkende Aufgabe, die die LEDs in einer Schleife ein- und ausschaltete, unterbrochen. Diese Tasks sind Teil des FreeRTOS-Kernels (man kann sie sich auch als Threads vorstellen), die eine einzelne Aufgabe in einer Schleife ausführen sollen. Der FreeRTOS-Kernel gibt Mikrocontroller-Anwendungen die Möglichkeit, den Prozessor zu optimieren, indem einzelne Tasks zur Ausführung eingeplant werden, sobald sich eine andere Task in einen suspendierten oder blockierten Zustand versetzt hat (z.B. mit **vTaskSuspend** oder **vTaskDelay**). Erfahren Sie [hier](https://www.freertos.org/implementation/a00005.html) mehr über das Scheduling mit FreeRTOS.

{{% notice info %}}
Um den seriellen Monitor zu verlassen, drücken Sie **STRG** + **]**.
{{% /notice %}}

## Aufräumen
Um die genutzten Ressourcen zu optimieren und unerwünschte Gebühren für den AWS-Cloud-Service zu vermeiden, löschen Sie den Flash-Speicher auf Ihrem Gerät, um ihn für die nachfolgenden Tutorials vorzubereiten. Um das Flash zu löschen, müssen Sie sich in einem bereits erstellten Projekt befinden (z. B. in dem, das Sie gerade abgeschlossen haben), einen laufenden seriellen Monitor beenden, der den Anschluss blockieren könnte (drücken Sie **STRG** + **]**), und den Befehl verwenden:

```
idf.py erase_flash -p <<DEVICE_PORT>>
```

## Fazit
Hoffentlich hat Ihnen der Aufbau Ihres Cloud-verbundenen Blinky-Projekts gefallen. Mit der Toolchain von Espressif (ESP-IDF) konnten Sie die Firmware Ihres Core2 for AWS IoT EduKit erstellen, flashen und überwachen. Ihr Gerät ist jetzt als AWS IoT-Ding bei AWS für eine sichere Cloud-Konnektivität mit eingebetteten Gerätezertifikaten und privaten Schlüsseln, die das Gerät nie verlassen können, registriert. Sie haben eine Verbindung zu AWS IoT Core hergestellt, Nachrichten vom Gerät über MQTT gesendet und eine MQTT-Nachricht vom MQTT-Client der AWS IoT-Konsole empfangen, mit der Sie das Licht am Gerät umschalten können.


Weiter zum nächsten Abschnitt, dem [Smart Thermostat](/de/smart-thermostat.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}