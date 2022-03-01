+++
title = "Fazit"
weight = 50
pre = "<b>e. </b>"
+++
## Fazit
Sie haben dieses praktische Modul abgeschlossen, um Ihre Intelligente Thermostat-Anwendung zu aktualisieren und ein geschultes Modell für maschinelles Lernen zu nutzen. Sie haben jetzt einen Einblick in den kompletten Ablauf des Erstellens einer IoT-Anwendung gesehen: Erfassen Sie physische Daten von Sensoren, synchronisieren Sie den Status zwischen Edge und Cloud, erstellen Sie eine serverlose Anwendung zur Steuerung eines Edge-Geräts und wandeln Sie Daten mit maschinellem Lernen in nützliche Erkenntnisse um.

Wenn Sie nach Ideen suchen, wie Sie die Lösung erweitern können, finden Sie hier einige:

* Fügen Sie dem IoT Analytics-Datensatz einen Timer hinzu, sodass er einmal täglich eine aktualisierte Inhaltsdatei erstellt.
* Erstellen Sie eine einfache Webbenutzeroberfläche oder stellen Sie Amazon QuickSight bereit, um den in IoT Analytics erstellten Datensatz zu visualisieren.
* Fügen Sie in IoT Analytics eine neue Pipeline-Aktivität hinzu, um das lokale Wetter über eine Lambda-Funktion abzurufen und es zu Ihren Thermostat-Daten hinzuzufügen.
* Verwenden Sie die Bibliotheken der Core2 des AWS IoT EduKit, um bei jeder Änderung des HVAC-Status einen Sound zu erzeugen.
* Verwenden Sie die Bibliotheken der Core2 des AWS IoT EduKit, um die LED-Streifen zu blinken, wenn sich die Raumbelegung ändert.

## Bereinigen
Zwischen dieser und der vorherigen Lösung (**Intelligentes Thermostat**) haben Sie die folgenden Ressourcen in AWS erstellt:

* IoT-Core Regel.
* IAM-Rollen.
* Detektor-Modell in IoT-Events.
* IoT-Analytics-Projekt (Kanal, Pipeline, Datenspeicher, Datensatz).
* Lambda-Funktion.
* SageMaker
* S3-Bucket (Teil des SageMaker Studio-Projekts).
* SageMaker-Modellgruppe.
* SageMaker-Endpunkt.

Für Ihr Konto fallen weiterhin auf drei Arten gemessene Gebühren an. Erstens für bereitgestellte Rechenressourcen wie den SageMaker-Endpunkt, der auf einer virtuellen Instanz gehostet wird. Für diese Ressourcen fallen die Gebühren stundenweise an. Zweitens für die Speicherung von Daten in Ressourcen wie S3-Buckets und Ihrem IoT-Analytics-Projekt (Kanal, Datenspeicher und verarbeitete Datensatzergebnisse). Diese Ressourcen haben Abrechnungsdimensionen wie Gigabyte-Monat. Drittens werden Ihnen ereignisgesteuerte Aktivitäten wie das Veröffentlichen von Nachrichten im IoT Core, das Aufrufen der Lambda-Funktion und das Verarbeiten von Ereignissen in IoT-Ereignissen pro Ereignis in Rechnung gestellt. Wenn Sie also Ihr Gerät ausschalten würden, würde die ereignisgesteuerte Aktivität eingestellt. Sie sollten diese AWS-Ressourcen vernichten, wenn Sie mit der Lösung fertig sind und nicht beabsichtigen, sie weiter zu verwenden.

Der SageMaker-Endpunkt ist die teuerste Ressource, da er stündlich abgerechnet wird. Wir empfehlen Ihnen, diese Ressource zu vernichten, wenn Sie sie nicht mehr verwenden. Ihr Modell selbst bleibt bestehen und könnte später erneut bereitgestellt werden. So löschen Sie den SageMaker-Endpunkt:

1. Gehen Sie zur SageMaker-Verwaltungskonsole, wählen Sie Endpoints, suchen Sie Ihren Endpoint in der Liste und löschen Sie ihn.

So zerstören Sie IoT Analytics-Speicherressourcen:

1. Gehen Sie zu AWS IoT Analytics, wählen Sie Kanäle, suchen Sie Ihren Kanal in der Liste und löschen Sie ihn.
2. Gehen Sie zu AWS IoT Analytics, wählen Sie Datenspeicher, suchen Sie Ihren Datenspeicher in der Liste und löschen Sie ihn.
3. Gehen Sie zu AWS IoT Analytics, wählen Sie Datensätze, suchen Sie Ihren Datensatz in der Liste und löschen Sie ihn.

Für die IoT-Core-Regel, die Lambda-Funktion, die Eingabe und das Detektor-Modell in IoT Events sowie die IoT-Analytics-Pipeline fallen nur weitere Gebühren an, wenn Sie in Gebrauch sind. Wenn Sie sie nicht mehr benötigen, können Sie sie löschen, indem Sie sie von den entsprechenden Ressourcendetailseiten in ihren jeweiligen Konto löschen:

1. Gehen Sie zu AWS IoT Core, wählen Sie Handeln, wählen Sie Regeln, suchen Sie Ihre Regeln in der Liste und löschen Sie sie.
2. Gehen Sie zu AWS IoT Analytics, wählen Sie Pipelines, suchen Sie Ihre Pipeline in der Liste und löschen Sie sie.
3. Gehen Sie zu AWS Lambda, wählen Sie Funktionen, suchen Sie Ihre Funktion in der Liste und löschen Sie sie.
4. Gehen Sie zu AWS IoT-Ereignisse, wählen Sie Eingaben, suchen Sie Ihre Eingabe in der Liste und löschen Sie sie.
5. Gehen Sie zu AWS IoT-Ereignisse, wählen Sie Detector-Modelle, suchen Sie Ihr Modell in der Liste und löschen Sie es.


Gehen Sie zum nächsten Tutorial, [**Einführung in Alexa für IoT**](/de/intro-to-alexa-for-iot.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}