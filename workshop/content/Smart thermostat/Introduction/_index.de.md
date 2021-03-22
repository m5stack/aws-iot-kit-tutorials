+++
title = "Einführung"
weight = 10
pre = "<b>a. </b>"
+++

## Ihre Aufgabe
In diesem Szenario übernehmen Sie die Rolle eines Full-Stack-Entwicklers, der die Aufgabe hat, Thermostatfunktionen eines Besprechungsraums zu automatisieren, um Energie zu sparen. Sie verwenden die "Core2 for AWS IoT EduKit"-Referenzhardware für die Thermostat-Hardware und stellen eine End-to-End-Lösung bereit, die das Referenzhardware-Kit als HLK-Controller mit der Leistung der AWS-Cloud kombiniert. Wir gehen von einem fiktiven HLK-System aus und schließen die Lösung am Rand mit Ihrem "Core2 for AWS IoT EduKit" als Thermostat ab.

Der Thermostat sollte einen kleineren Temperaturbereich verwenden, wenn die Mitarbeiter den Raum besetzen, um deren Komfort zu maximieren. Wenn der Raum nicht besetzt ist, wird ein breiterer Temperaturbereich zugelassen, um Energie zu sparen. Die Lösung sollte erkennen, wenn Mitarbeiter im Raum anwesend sind und die HLK einschalten, um die Komfortzone der Temperatur zu liefern.

## Problemlösung
Um die Kosten für eine Kameraausrüstung zu minimieren, die erkennt, wann der Raum belegt ist, wird bei dieser Lösung das Mikrofon des Kits verwendet, um einen einfachen Geräuschpegel zu erfassen. Sie erfassen den Umgebungsgeräuschpegel über das Mikrofon und die Raumtemperatur über die Sensoren des Geräts und veröffentlichen diese Werte dann bei AWS.

Ihre serverlose Lösung in AWS wandelt den Geräuschpegel in einen Booleschen Wert um und verwendet diesen, um zu bestimmen, ob der Raum belegt ist. Sie können einen einfachen Schwellenwert des Geräuschpegels verwenden, um diese Bestimmung vorzunehmen.

Wenn der Raum belegt ist und die gemessene Temperatur außerhalb der Komfortgrenzen liegt, sendet die Lösung einen Befehl an das Gerät, um je nach Bedarf die Heizung oder Kühlung zu starten. Wenn die gemessene Temperatur dann innerhalb der Grenzen liegt, sendet die Lösung einen Befehl an das Gerät, um den Standby-Modus wieder aufzunehmen.

## Architektur
![Smarter Thermostat Lösungsarchitektur](introduction/thermostat-overview.png)

## Los geht's!
Sind Sie bereit, mit dem Bau zu beginnen? Lassen Sie uns überprüfen, ob Sie die folgenden Voraussetzungen erfüllt haben:

1. Haben Sie das **Blinky Hello World-Tutorial** abgeschlossen?
2. Ist Ihr "Core2 for AWS IoT EduKit" bereits in AWS IoT Core bereitgestellt? Das heißt, es gibt ein registriertes AWS IoT-Ding mit einem Zertifikat und einer angehängten Richtlinie, die Veröffentlichungs- und Anmeldevorgänge ermöglicht? Diese Schritte finden Sie im **Blinky Hello World-Tutorial**.
3. Haben Sie bestätigt, dass Sie mit einem Test-MQTT-Client wie dem in der AWS IoT Core-Konsole die von Ihrem Gerät eingehenden Nachrichten sehen können? Sie sollten in der Lage sein, ein Topic zu abonnieren, das Ihr Gerät veröffentlicht, und sehen, dass diese Nachrichten im Test-Client ankommen.
4. Wissen Sie, an welchem seriellen Port Ihr "Core2 for AWS IoT EduKit"-Gerät angeschlossen ist? Dies wird auch im Blinky Hello World-Tutorial behandelt. Sie werden es in mehrere der Schritte dieses Tutorials für Platzhalterwerte wie **<<DEVICE_PORT>>** einfügen.

Wenn ja, lassen Sie uns mit dem nächsten Kapitel, der Datenerfassung, fortfahren.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}