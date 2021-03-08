+++
title = "Intelligenter Raum"
chapter = true
weight = 40
pre = "<b>4. </b>"
+++

In diesem Lernprogramm werden Sie die intelligente Thermostat-Lösung aus dem vorherigen Modul, **Intelligentes Thermostat**, zu einer intelligenten Raum-Lösung erweitern. Ein Intelligenter Raum ist das Konzept, Erkenntnisse über einen Raum zu nutzen, um den Raum zu verbessern oder ihm weitere Funktionen zu verleihen. Sie werden Analyse- und maschinelle Lernfunktionen nutzen, um Vorhersagen aus Rohdaten über die Raumbelegung abzuleiten, in denen Ihr intelligenter Thermostat eingesetzt wird. Die intelligente Raum-Lösung leitet Sie an, wie Sie aus Ihren Thermostatdaten ein neues Modell für maschinelles Lernen erstellen und die Klassifizierung der Raumbelegung verbessern können, um Ihr Intelligentes Thermostat noch besser zu bedienen.

Annahmen. Überprüfen Sie vor Beginn dieses Labors die folgenden Voraussetzungen:
1. Sie haben ein M5Stack Core2 ESP32 IoT Development Kit für AWS IoT EduKit.
2. Sie haben ein AWS-Konto, auf dem keine Produktivworkloads ausgeführt werden (d. h. ein Konto, das für Sandbox- und Evaluierungszwecke sicher ist).
3. Sie haben eine Benutzeranmeldung oder eine Rolle für das AWS-Konto mit Administratorzugriff.
4. Ihr Core2 for AWS IoT EduKit wurde in AWS IoT Core bereitgestellt und kommuniziert bereits mit AWS über MQTT. Beginnen Sie mit dem [**Intelligentes Thermostat**](http://localhost:1313/en/smart-thermostat.html) Tutorial, wenn Sie diese Basisfunktionalität noch nicht abgeschlossen haben.

Lernziele. Am Ende dieses Labs sollten Sie wissen:

1. Wie Sie Gerätetelemetrie an AWS IoT Analytics weiterleiten, um sie zu speichern, umzuwandeln und analytische Datensätze zu erstellen.
2. Wie Sie Amazon SageMaker Studio verwenden, um ein Experiment zu erstellen, das automatisch ein Modell für maschinelles Lernen aus Ihren Datensätzen generiert.
3. Wie sie ein Modell für maschinelles Lernen mit einen API-Endpunkt bereitstellen.
4. Wie eine serverlose Funktion eine ML-Inferenz-API aufruft, um eine Anwendung zu erweitern.

Beginnen wir mit der [**Einführung**](/de/smart-spaces/introduction.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}