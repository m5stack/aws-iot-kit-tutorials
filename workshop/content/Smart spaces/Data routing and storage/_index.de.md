+++
title = "Datenweiterleitung und -speicherung"
weight = 20
pre = "<b>b. </b>"
+++
## Einführung in das Kapitel
Am Ende dieses Kapitels sollte Ihre serverlose Anwendung Folgendes tun:

* Weiterleiten von Nachrichten, die vom intelligenten Thermostat empfangen wurden, an einen verwalteten Speicher- und Analysedienst.
* Sie können eine Abfrage Ihrer verarbeiteten Daten ausführen, um eine materialisierte Ansicht der Ergebnisse zu erstellen.

## Konzepte zum Speichern und Analysieren von IoT-Daten
Bis zu diesem Zeitpunkt war jeder Aspekt Ihrer IoT-Lösung kurzlebig in dem Sinne, dass jede Nachricht empfangen, verarbeitet und dann verworfen wurde. Im Fall des IoT-Events Detektor-Modells gibt es eine status-behaftete Entität, die auf neue Nachrichten reagiert, aber ansonsten ist kein Verlauf Ihrer Nachrichten gespeichert. Nun folgt der Schritt, der einen Datenspeicher für Ihre Thermostat-Nachrichten einrichtet.

AWS bietet viele Möglichkeiten, Daten in der Cloud zu speichern, auch zum Speichern von IoT-Daten. Diese Lösung befürwortet die Verwendung eines Dienstes namens [AWS IoT Analytics](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html), bei dem es sich um einen verwalteten Service handelt, um IoT-Daten in großen Mengen zu empfangen, zu speichern, zu verarbeiten und zu analysieren. Sie verwenden IoT Analytics, um eine Teilmenge jedes Updates des Geräteschattens zu speichern.

Die AWS IoT Analytics-Dokumentation behandelt dies ausführlicher, hier ist nur eine kurze Zusammenfassung der Funktionsweise. Der Einstiegspunkt in den Dienst ist eine Ressource, die als Kanal bezeichnet wird. Ein Kanal speichert alle Rohdaten für Ihren Workflow. Es sendet auch eine Kopie jeder empfangenen Nachricht an die nächste Ressource, die als Pipeline bezeichnet wird. Eine Pipeline besteht aus einer Reihe von Aktivitäten, die Daten verarbeiten, bereinigen, filtern und anreichern, bevor sie in Analyseanwendungsfällen verwendet werden. Die verarbeiteten Nachrichten werden von der Pipeline in einen Datenspeicher gesendet. Ein Datenspeicher, wie ein Kanal, ist eine langlebige Speichereinheit für verarbeitete Daten. Schließlich ist ein Datensatz die letzte Ressource im AWS IoT Analytics-Projekt. Ein Datensatz definiert eine SQL-ähnliche Abfrage, die Nachrichten aus einem Datenspeicher als materialisierte Ansicht lesen und den Inhalt der Abfrage an ein Ziel wie einen S3-Bucket liefern kann.

Ihre Intelligente-Raum-Lösung sammelt mehrere Stunden an Laufzeitdaten von Ihrem Thermostat in AWS IoT Analytics. Danach werden diese Daten durch das Ergebnis einer Datensatzabfrage für die Verwendung in unserer von Amazon SageMaker bereitgestellten Toolchain für maschinelles Lernen verfügbar gemacht.

[AWS IoT Analytics](https://aws.amazon.com/iot-analytics/) bietet noch viel mehr, aber für die Zwecke dieses Moduls ist es der einfachste Weg, unsere Thermostat-Meldungen über die Zeit zu speichern und als Trainingsdatensatz für unser Modell des maschinellen Lernens zu aggregieren.

{{% notice warning %}}
Wenn diese Anwendung länger als 6 Stunden ausgeführt wird, kann dies zu AWS-Gebühren führen. Es wird empfohlen, dieses Tutorial nach dieser Zeitspanne zu beenden und die [Bereinigungsschritte](/de/smart-spaces/conclusion.html#clean-up) durchzuführen, um unerwünschte Kosten zu vermeiden.
{{% /notice %}}

## So richten Sie die Infrastruktur ohne Server ein
In den folgenden Schritten wird beschrieben, wie Sie eine neue IoT-Core-Regel und ein neues AWS IoT Analytics-Projekt erstellen und mithilfe der Regel Geräteschattenmeldungen an Ihr AWS IoT Analytics-Projekt weiterleiten. Im Assistenten zur Erstellung von IoT-Core Regeln gibt es eine praktische Oberfläche, mit der Sie das gesamte AWS IoT Analytics-Projekt automatisiert erstellen lassen können!

1. Wählen Sie in der [AWS IoT Core Console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/) **Handeln**, dann **Regeln** und schließlich **Erstellen**.
2. Geben Sie Ihrer Regel einen Namen wie *StoreIoTAnalytics* und eine Beschreibung.
3. Verwenden Sie die folgende Abfrage. Ersetzen Sie **<<CLIENT_ID>>** unbedingt durch die Client-ID/Seriennummer, die auf dem Bildschirm Ihres Core2 des AWS IoT Edukit aufgedruckt ist.

```SQL
SELECT current.state.reported.sound, current.state.reported.temperature, current.state.reported.hvacStatus, current.state.reported.roomOccupancy, timestamp FROM '$aws/things/<<CLIENT_ID>>/shadow/update/documents'
```

4. Wählen Sie **Aktion hinzufügen**.
5. Wählen Sie *Eine Nachricht an IoT Analytics senden* und wählen Sie **Aktion konfigurieren**.
6. Wählen Sie *IoT Analytics-Ressourcen schnell erstellen* und geben Sie einen Projektnamen für *Ressourcenpräfix* an. Weitere Modulschritte gehen davon aus, dass das Präfix „smartspace“ ist. Wählen Sie **Schnellerstellung** und alle Ihre AWS IoT Analytics-Ressourcen werden automatisch erstellt und konfiguriert.
7. Wählen Sie **Aktion hinzufügen**, um diese Aktion zu konfigurieren und zum Formular zur Regelerstellung zurückzukehren.
8. Wählen Sie **Regel erstellen**, um Ihre neue Regel zu erstellen.

## Schritte zur Validierung
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihre serverlose Anwendung wie vorgesehen konfiguriert ist:

1. Stellen Sie sicher, dass Ihr intelligentes Thermostat eingeschaltet ist, Daten veröffentlicht und in dem Raum eingesetzt wird, in dem Sie trainieren möchten.
2. Prüfen Sie mithilfe der AWS IoT Analytics-Konsole die neuesten Datensatzinhalte und stellen Sie sicher, dass historische Aufzeichnungen über Geräuschpegel, Temperatur und Raumbelegung vorliegen. Um dies zu überprüfen, suchen Sie Ihren Datensatz in der [IoT Analytics-Konsole](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets), wählen Sie **Aktionen** und **Jetzt ausführen** und warten Sie dann, bis die *Ergebnisvorschau* mit den neuesten Inhalten aktualisiert wird. Sie sollten ähnliche Ergebnisse wie die folgenden sehen:

{{< img "iot_analytics-dataset_run.en.png" "Running the data set" "1 - Aktion, 2 - Jetzt ausführen" >}}
{{< img "iot_analytics-dataset_preview.en.png" "Preview of the data set" >}}

Wenn diese wie erwartet funktionieren, gehen wir zu [**Machinelles Lernen**](/de/smart-spaces/machine-learning.html) über.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}