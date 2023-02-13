+++
title = "Einführung"
weight = 10
pre = "<b>a. </b>"
+++

## Deine Aufgabe
In diesem Szenario übernehmen Sie die Rolle eines Full-Stack-Entwicklers, der mit der Automatisierung der Thermostatfunktionen eines Besprechungsraums beauftragt ist, um Energie zu sparen. Sie verwenden den Core2 des AWS IoT Kit als Thermostat-Hardware und stellen eine End-to-End-Lösung bereit, die das Referenzhardware-Kit als HLK-Controller an die Leistung der AWS-Cloud anbindet. Wir erstellen ein fiktives HLK-System, in dem Ihr Core2 des AWS IoT Kit als Thermostat fungiert.

Das Thermostat sollte einen begrenzten Temperaturbereich verwenden, wenn Mitarbeiter den Raum belegen, um ihren Komfort zu maximieren. Wenn der Raum nicht belegt ist, ist ein größerer Bereich zulässig, um Energie zu sparen. Die Lösung sollte erkennen, wann Mitarbeiter im Raum anwesend sind, und die HLK einsetzen, um angenehme Temperaturen zu gewährleisten.

## Probleme lösen
Anstatt Kameraausrüstung zu verwenden, um zu erkennen, wann der Raum belegt ist, minimiert diese Lösung die Kosten, indem das Kit-Mikrofon zur Messung des Geräuschpegels verwendet wird. Sie erfassen den Geräuschpegel der Umgebung mit dem Mikrofon und die Raumtemperatur mit den Sensoren des Geräts. Diese senden Sie dann an AWS IoT Core.

Ihre Serverless-Lösung in AWS wandelt den Geräuschpegel in einen Booleschen Wert (wahr oder falsch) um und ermittelt anhand dessen, ob der Raum belegt ist. Sie können einen einfachen Schwellwert verwenden, um diese Bestimmung zu treffen.

Während der Raum belegt ist und die gemessene Temperatur außerhalb der Komfortgrenzen liegt, sendet die Lösung einen Befehl an das Gerät, um nach Bedarf mit dem Heizen oder Kühlen zu beginnen. Wenn die Temperatur dann innerhalb der Grenzen gemessen wird, sendet die Lösung einen Befehl an das Gerät, um den Standby-Modus fortzusetzen.

## Architektur der Lösung
![Intelligente Thermostatlösungsarchitektur](introduction/thermostat-overview.png)

## Lass uns starten!
Sind Sie bereit, mit dem Bauen zu beginnen? Stellen wir sicher, dass Sie die folgenden Voraussetzungen erfüllt haben:

1. Hast du das Tutorium **Blinky Hello World** abgeschlossen?
2. Ist Ihr Core2 des AWS IoT Kit bereits in AWS IoT Core registriert? Das bedeutet, dass Sie es als AWS IoT-Objekt mit einem Zertifikat und einer angehängten Richtlinie registriert haben, die Veröffentlichungs- und Abonnementvorgänge ermöglicht? Diese Schritte finden Sie im Tutorial **Blinky Hello World**.
3. Haben Sie bestätigt, dass Sie mit einem Test-MQTT-Client (wie dem in der AWS IoT Core-Konsole) Nachrichten von Ihrem Gerät einsehen können? Sie sollten in der Lage sein, ein Thema zu abonnieren, auf dem Ihr Gerät veröffentlicht, und sehen, dass diese Nachrichten im Testclient eingehen.
4. Wissen Sie, an welcher seriellen Schnittstelle Ihr Core2 des AWS IoT Kit-Gerät montiert ist? Dies wird auch im Tutorium **Blinky Hello World** behandelt.

Wenn ja, fahren wir mit dem nächsten Kapitel [**Datenerfassung**](/de/smart-thermostat/data-acquisition.html) fort.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}