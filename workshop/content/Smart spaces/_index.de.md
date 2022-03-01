+++
title = "Intelligenter Raum"
chapter = true
weight = 40
pre = "<b>4. </b>"
+++

In diesem Tutorial erweitern Sie die Lösung vom vorherigen Modul, **Intelligentes Thermostat**, zu einer Intelligenten Raum-Lösung. Ein intelligenter Raum ist das Konzept, Erkenntnisse über einen Raum zu nutzen, um den Raum zu erweitern oder ihm andere Möglichkeiten zu bieten. Sie verwenden Analysen und Funktionen des maschinellen Lernens, um aus Rohdaten Vorhersagen über die Raumbelegung abzuleiten, in dem ihr Thermostat eingesetzt wird. Diese Lösung zeigt Ihnen, wie Sie aus Ihren Daten ein neues Modell für maschinelles Lernen erstellen und wie Sie die Klassifizierung der Raumbelegung verbessern können, um Ihr intelligentes Thermostat noch besser bedienen zu können.

Annahmen. Bevor Sie mit diesem Tutorial beginnen, überprüfen Sie die folgenden Voraussetzungen:
1. Sie haben ein M5Stack Core2 ESP32 IoT-Entwicklungskit für AWS IoT EduKit.
2. Sie haben ein AWS-Konto, das keine Produktionsworkloads ausführt (d. h. ein Konto, das für Sandbox- und Evaluierungszwecke eingerichtet ist).
3. Sie haben einen Nutzer oder eine Benutzerrolle für das AWS-Konto mit Administratorzugriff.
3. Ihr Core2 für AWS IoT EduKit wurde in AWS IoT Core bereitgestellt und kommuniziert bereits über MQTT mit AWS. Beginnen Sie mit der [**Intelligenten Thermostat**](/de/smart-thermostat.html) -Lerneinheit, wenn Sie diese Funktionsgrundlinie noch nicht abgeschlossen haben.

Ziele. Am Ende dieser Einheit sollten Sie wissen:
1. So leiten Sie die Gerätetelemetrie zur Speicherung, Transformation und Erstellung analytischer Datensätze an AWS IoT Analytics weiter.
2. So verwenden Sie Amazon SageMaker Studio, um ein Experiment zu erstellen, das automatisch aus Ihren Datensätzen ein Modell für maschinelles Lernen generiert.
3. So stellen Sie ein Modell für maschinelles Lernen auf einem erreichbaren API-Endpunkt bereit.
4. Sie verwenden eine serverlose Funktion mit der ML-Inferenz-API, um die Anwendung zu erweitern.

Beginnen wir mit der [**Einführung**](/de/smart-spaces/introduction.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}