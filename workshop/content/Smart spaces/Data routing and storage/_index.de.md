+++
title = "Datenweiterleitung und -speicherung"
weight = 20
pre = "<b>b. </b>"
+++

## Einleitung
Am Ende dieses Kapitels sollte Ihre serverlose Anwendung Folgendes tun:

- Weiterleitung der vom intelligenten Thermostat empfangenen Meldungen an einen verwalteten Speicher- und Analysedienst
- Abfragen zu Ihren verarbeiteten Daten auszuführen, um eine materialisierte Ansicht der Ergebnisse zu erzeugen

## Konzepte zum Speichern und Analysieren von IoT-Daten
Bis zu diesem Punkt war jeder Aspekt Ihrer IoT-Lösung flüchtig in dem Sinne, dass jede Nachricht empfangen, verarbeitet und dann verworfen wurde. Im Fall des IoT-Ereignis-Detektormodells gibt es zwar eine zustandsabhängige Entität, die auf neue Nachrichten reagiert, aber ansonsten gibt es keine gespeicherte Historie Ihrer Thermostatnachrichten. Dies ist der Schritt, der einen Datenspeicher für Ihre Thermostatmeldungen einrichtet.

AWS bietet viele Möglichkeiten zum Speichern von Daten in der Cloud, und das Speichern von IoT-Daten ist nicht anders. In dieser Lösung bevorzugen wir die Verwendung eines Dienstes namens [AWS IoT Analytics](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html), der ein verwalteter Service ist, der speziell für den Empfang, die Speicherung, Verarbeitung und Analyse von IoT-Daten in großen Mengen entwickelt wurde. Sie werden IoT Analytics verwenden, um eine Teilmenge jeder Aktualisierung des Geräteschattens zu speichern.

In der Dokumentation zu AWS IoT Analytics wird dies ausführlicher behandelt, aber hier ist eine kurze Zusammenfassung der Funktionsweise. Der Einstiegspunkt in den Service ist eine Ressource namens Kanal. Ein Kanal speichert alle Rohdaten für Ihren Workflow. Er sendet außerdem eine Kopie jeder empfangenen Nachricht an die nächste Ressource, die Pipeline genannt wird. Eine Pipeline ist eine Reihe von Aktivitäten zum Verarbeiten, Bereinigen, Filtern und Anreichern von Daten, bevor sie in Analyseanwendungsfällen verwendet werden. Die verarbeiteten Nachrichten werden von der Pipeline in einem Datenspeicher abgelegt. Ein Datenspeicher ist, wie ein Kanal, eine langlebige Speichereinheit für verarbeitete Daten. Schließlich ist ein Datensatz die letzte Ressource im AWS IoT Analytics-Projekt. Ein Datensatz definiert eine SQL-ähnliche Abfrage, die Nachrichten aus einem Datenspeicher als materialisierte Ansicht auslesen und den Inhalt der Abfrage an ein Ziel wie einen S3-Bucket liefern kann.

Ihre intelligente Raum Lösung sammelt mehrere Stunden Laufzeitdaten von Ihrem Thermostat in AWS IoT Analytics. Danach wird das Ergebnis einer Datensatzabfrage diese Daten für die Verwendung in unserer von Amazon SageMaker bereitgestellten Toolchain für maschinelles Lernen verfügbar machen.

Es gibt noch viel, viel mehr zu [AWS IoT Analytics](https://aws.amazon.com/iot-analytics/), aber für die Zwecke dieses Moduls ist es die einfachste Möglichkeit, den Verlauf unserer Thermostatmeldungen zu speichern und sie als Trainingsdatensatz für unser maschinelles Lernmodell zu aggregieren.

{{% notice warning %}}
Wenn Sie diese Anwendung länger als 6 Stunden laufen lassen, können AWS-Gebühren anfallen. Es wird empfohlen, dieses Lernprogramm in dieser Zeit zu beenden und die [Bereinigungsschritte](/de/smart-spaces/conclusion.html#Bereinigungsschritte) durchzuführen, um unerwünschte Kosten zu vermeiden.
{{% /notice %}}

## So richten Sie die serverlose Infrastruktur ein
In den folgenden Schritten wird beschrieben, wie Sie eine neue IoT Core-Regel und ein neues AWS IoT Analytics-Projekt erstellen und die Regel zur Weiterleitung von Geräteschattenmeldungen an Ihr AWS IoT Analytics-Projekt verwenden. Es gibt eine praktische Schnittstelle im Assistenten zum Erstellen von IoT Core-Regeln, mit der Sie das gesamte AWS IoT Analytics-Projekt in Ihrem Namen erstellen können!

1. Wählen Sie in der [AWS IoT Core-Konsole](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/) zuerst **Act**, dann **Rules** und dann **Create**.
2. Geben Sie Ihrer Regel einen Namen wie *storeInIoTAnalytics* und eine Beschreibung.
3. Verwenden Sie die folgende Abfrage. Stellen Sie sicher, dass Sie die **<<CLIENT_ID>>** durch die Client-ID/Seriennummer ersetzen, die auf dem Bildschirm Ihres Core2 for AWS IoT Edukit-Referenz-Hardware-Kits aufgedruckt ist.


```SQL
SELECT current.state.reported.sound, current.state.reported.temperature, current.state.reported.hvacStatus, current.state.reported.roomOccupancy, timestamp FROM '$aws/things/<<CLIENT_ID>>/shadow/update/documents'
```

4. Wählen Sie **Add action**, um *Set one or more actions*.
5. Wählen Sie *Send a message to IoT Analytics* und wählen Sie **Configure action**.
6. Wählen Sie *Quick create IoT Analytics resources* und geben Sie einen Projektnamen für das *Resource prefix* an. Die weiteren Modulschritte gehen davon aus, dass das Präfix `smartspace` lautet. Wählen Sie **Quick Create** und alle Ihre AWS IoT Analytics-Ressourcen werden automatisch erstellt und konfiguriert.
7. Wählen Sie **Add action**, um die Konfiguration dieser Aktion abzuschließen und zum Formular für die Regelerstellung zurückzukehren.
8. Wählen Sie **Create rule**, um das Erstellen Ihrer neuen Regel abzuschließen.

## Validierungsschritte

Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihre serverlose Anwendung wie vorgesehen konfiguriert ist:

1. Vergewissern Sie sich, dass Ihr intelligenter Thermostat eingeschaltet ist, Daten veröffentlicht und in dem Raum eingesetzt wird, für den Sie trainieren möchten.
2. Überprüfen Sie mithilfe der AWS IoT Analytics-Konsole den neuesten Inhalt des Datensatzes und vergewissern Sie sich, dass historische Aufzeichnungen zu Umgebungsgeräuschpegel, Temperatur und Raumbelegung vorhanden sind. Um dies zu überprüfen, suchen Sie Ihren Datensatz in der [IoT Analytics-Konsole](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets), wählen Sie **Actions** und **Run now** und warten Sie dann, bis die *Ergebnisvorschau* mit dem neuesten Inhalt aktualisiert wurde. Sie sollten ähnliche Ergebnisse wie die folgenden sehen:

{{< img "iot_analytics-dataset_run.en.png" "Running the data set" >}}
{{< img "iot_analytics-dataset_preview.en.png" "Preview of the data set" >}}

Wenn diese wie erwartet funktionieren, lassen Sie uns mit dem [maschinellen Lernen fortfahren](/de/smart-spaces/machine-learning.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}