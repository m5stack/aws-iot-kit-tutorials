+++
title = "Fazit"
weight = 50
pre = "<b>e. </b>"
+++

## Fazit
Sie haben dieses Praxismodul abgeschlossen, um Ihre intelligente Thermostat Anwendung zu aktualisieren und die Vorteile eines trainierten maschinellen Modells zu nutzen. Sie haben nun einen Einblick in die End-to-End-Erfahrung beim Erstellen einer IoT-Anwendung erhalten: Erfassen von physischen Daten von Sensoren, Synchronisieren des Zustands zwischen dem Edge und der Cloud, Erstellen einer serverlosen Anwendung zur Steuerung eines Edge-Geräts und Umwandeln von Daten in verwertbare Erkenntnisse mit maschinellem Lernen.

Wenn Sie nach Ideen suchen, wie Sie die Lösung weiter ausbauen können, finden Sie hier ein paar:

- Fügen Sie dem IoT-Analytics-Datensatz einen Timer hinzu, damit er einmal pro Tag eine aktualisierte Inhaltsdatei erstellt
- Erstellen Sie eine einfache Web-UI oder stellen Sie Amazon QuickSight bereit, um den in IoT Analytics erstellten Datensatz zu visualisieren
- Fügen Sie eine neue Pipeline-Aktivität in IoT Analytics hinzu, um das lokale Wetter über eine Lambda-Funktion abzurufen und es zu Ihren Thermostatdaten hinzuzufügen
- Verwenden Sie die Bibliotheken der Core2 for AWS IoT EduKit-Referenzhardware, um einen Ton auszugeben, wenn sich der hvacStatus ändert
- Verwenden Sie die Bibliotheken der Core2 for AWS IoT EduKit-Referenzhardware, um die LED-Streifen bei jeder Änderung der Raumbelegung blinken zu lassen

## Aufräumen
Zwischen dieser Lösung und der vorherigen (**Intelligentes Thermostat**) haben Sie die folgenden Ressourcen in AWS erstellt:

- IoT-Core Regeln
- IAM-Rollen
- IoT-Events Detektor-Modell
- IoT-Analytics-Projekt (Kanal, Pipeline, Datenspeicher, Datensatz)
- Lambda-Funktion
- SageMaker Studio
- S3-Bucket (Teil des SageMaker Studio-Projekts)
- SageMaker-Modellgruppe
- SageMaker-Endpunkt

Für Ihr Konto fallen weiterhin auf drei Arten Gebühren an. Erstens für bereitgestellte Rechenressourcen wie den SageMaker-Endpunkt, der auf einer virtuellen Instanz gehostet wird. Für diese Ressourcen fallen stündliche Gebühren an. Zweitens für die Speicherung von Daten in Ressourcen wie S3-Buckets und Ihr IoT Analytics-Projekt (Kanal, Datenspeicher und verarbeitete Datensatzergebnisse).
Diese Ressourcen haben Abrechnungsdimensionen wie Gigabyte-Monat. Drittens, für ereignisgesteuerte Aktivitäten wie das Veröffentlichen von Nachrichten an IoT Core, das Aufrufen der Lambda-Funktion und das Verarbeiten von Ereignissen in IoT Events, werden Sie pro Ereignis abgerechnet. Wenn Sie also Ihr Gerät abschalten würden, würde die ereignisgesteuerte Aktivität aufhören. Sie sollten jede dieser AWS-Ressourcen löschen, wenn Sie mit dem Lab fertig sind und nicht beabsichtigen, sie weiter zu verwenden.

Der SageMaker-Endpunkt ist die teuerste Ressource, da er stündlich berechnet wird. Es wird empfohlen, diese Ressource zu löschen, wenn Sie sie nicht mehr verwenden. Ihr Modell selbst bleibt bestehen und könnte später erneut eingesetzt werden. Schritte zum Löschen des SageMaker-Endpunkts:

1. Gehen Sie zur SageMaker-Verwaltungskonsole, wählen Sie Endpunkte, suchen Sie Ihren Endpunkt in der Liste und löschen Sie ihn.

Schritte zur Löschung von IoT Analytics-Speicherressourcen:

1. Gehen Sie zu AWS IoT Analytics, wählen Sie Channels, suchen Sie Ihren Kanal in der Liste und löschen Sie ihn.
2. Gehen Sie zu AWS IoT Analytics, wählen Sie Datenspeicher, suchen Sie Ihren Datenspeicher in der Liste und löschen Sie ihn.
3. Gehen Sie zu AWS IoT Analytics, wählen Sie Datensätze, suchen Sie Ihren Datensatz in der Liste und löschen Sie ihn.

Für die IoT-Core-Regel, die Lambda-Funktion, den IoT-Events-Input und -Detektor sowie die IoT-Analytics-Pipeline fallen nur dann weitere Gebühren an, wenn Sie sie verwenden. Wenn Sie sie nicht mehr benötigen, können Sie sie zerstören, indem Sie sie auf ihren jeweiligen Ressourcendetailseiten in ihren jeweiligen Management-Konsolen löschen.

1. Gehen Sie zu AWS IoT Core, wählen Sie Act, wählen Sie Rules, suchen Sie Ihre Regeln in der Liste und löschen Sie sie.
2. Gehen Sie zu AWS IoT Analytics, wählen Sie Pipelines, suchen Sie Ihre Pipeline in der Liste und löschen Sie sie.
3. Gehen Sie zu AWS Lambda, wählen Sie Functions, suchen Sie Ihre Funktion in der Liste und löschen Sie sie.
4. Gehen Sie zu AWS IoT Events, wählen Sie Inputs, suchen Sie Ihren Input in der Liste und löschen Sie ihn.
5. Gehen Sie zu AWS IoT Events, wählen Sie Detector models, suchen Sie Ihr Modell in der Liste und löschen Sie es.

Gehen Sie zum nächsten Abschnitt, [Einführung von Alexa für IoT](/de/intro-to-alexa-for-iot.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}