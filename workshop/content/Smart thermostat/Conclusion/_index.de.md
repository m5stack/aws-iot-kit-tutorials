+++
title = "Zusammenfassung"
weight = 60
pre = "<b>f. </b>"
+++

## Zusammenfassung

Sie haben dieses AWS IoT EduKit Praxistutorial abgeschlossen: Sie haben Ihr Gerät als intelligenten Thermostat mit AWS verbunden und eine einfache Anwendung erstellt, um die Raumbelegung zu erkennen und Zustandsänderungen der HLK zu steuern.

Als nächsten Schritt sollten Sie das nächste Praxis-Tutorial in dieser Reihe, **Smart Spaces**, in Betracht ziehen:

Wenn Sie auf der Suche nach zusätzlichen Ideen sind, um mit Ihrer neuen Lösung zu experimentieren, denken Sie an eine Möglichkeit, das Kundenerlebnis zu verbessern. Wie könnten Sie zum Beispiel die Lösung so aktualisieren, dass die HLK-Einstellung für einen minimalen Timer aktiviert bleibt, anstatt sich auszuschalten, nachdem nur 10 Sekunden lang kein Umgebungsgeräusch zu hören war? Können Sie dies erreichen, ohne neuen Code in das Gerät zu pushen? (Tipp: Ja, das können Sie.)

## Aufräumen
In dieser Lösung haben Sie die folgenden Ressourcen in AWS erstellt:

* AWS IoT Core Regel
* IAM Rollen
* AWS IoT Events Inputs und Detektormodell

Für keine dieser Ressourcen fallen laufende Gebühren an, nur weil sie vorhanden sind. Es fallen nur Gebühren an, wenn die Ressourcen für die Verarbeitung neuer Nachrichten von Ihrem Gerät verwendet werden. Wenn Sie mit dem nächsten praktischen Lernprogramm **Smart Spaces** fortfahren, wird empfohlen, keine der Ressourcen aus diesem Lernprogramm zu zerstören.

Wenn Sie das Experimentieren mit der Lösung dieses Tutorials abgeschlossen haben und nicht beabsichtigen, das **Smart Spaces** Tutorial zu machen, können Sie diese Ressourcen terminieren.

1. Gehen Sie zu AWS IoT Core, wählen Sie Act, wählen Sie Rules, suchen Sie Ihre Regel in der Liste und löschen Sie sie.
2. Gehen Sie zu AWS IoT Events, wählen Sie Detector models, suchen Sie Ihr Modell in der Liste und löschen Sie es. Wählen Sie Inputs, suchen Sie Ihren Eingang in der Liste und löschen Sie ihn.
3. Gehen Sie zu IAM, wählen Sie Rollen, suchen Sie die Rollen für Ihre IoT Core-Regeln und Ihr IoT-Ereignis-Detektormodell in der Liste und löschen Sie sie.
4. Schalten Sie Ihr "Core2 for AWS IoT EduKit" Referenz-Hardware-Kit aus oder führen Sie den Befehl in Ihrer Shell aus dem Ordner **Smart-Thermostat** aus, um zu verhindern, dass das Gerät verbunden ist und Nachrichten an AWS IoT Core sendet:
```bash
idf.py erase_flash -p <<DEVICE_PORT>>
```

Der nächste Lernabschnitt ist [**Smart Spaces**](/de/smart-spaces.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}