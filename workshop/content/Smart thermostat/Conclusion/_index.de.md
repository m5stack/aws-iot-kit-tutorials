+++
title = "Zusammenfassung"
weight = 60
pre = "<b>f. </b>"
+++
## Fazit
Sie haben dieses praktische AWS IoT EduKit Tutorial abgeschlossen, das Ihr Gerät als intelligenter Thermostat mit AWS verbindet und eine einfache Anwendung bereitstellt, um Raumbelegungen zu erkennen und Änderungen des HLK-Zustands voranzutreiben.

Als nächsten Schritt sollten Sie zusammen mit dem nächsten praktischen Tutorial in dieser Serie, **Intelligenter Raum**, folgen:

Wenn Sie nach zusätzlichen Ideen suchen, um mit Ihrer neuen Lösung zu experimentieren, überlegen Sie, wie Sie das Kundenerlebnis verbessern können. Wie können Sie beispielsweise die Lösung aktualisieren, sodass die HLK-Einstellung für eine Mindestzeit aktiviert bleibt, anstatt sich auszuschalten, nachdem nur 10 Sekunden lang keine Umgebungsgeräusche zu hören sind? Können Sie dies erreichen, ohne neuen Code auf Ihr Gerät zu übertragen? (Hinweis: ja, das kannst du.)

## Bereinigen
In dieser Lösung haben Sie die folgenden Ressourcen in AWS erstellt:

* IoT-Kernregel.
* IAM-Rollen.
* Eingabe- und Detektormodell für IoT-Ereignisse.

Für keine dieser Ressourcen fallen laufende gemessene Gebühren an, die nur durch vorhandene Ressourcen verursacht werden. Es fallen nur gemessene Gebühren an, wenn die Ressourcen zur Verarbeitung neuer Nachrichten von Ihrem Gerät verwendet werden. Wenn Sie mit dem nächsten praktischen Tutorial **Intelligenter Raum** fortfahren, empfehlen wir Ihnen, keine der Ressourcen aus diesem Tutorial zu vernichten.

Wenn Sie mit der Lösung dieses Tutorials fertig experimentiert haben und nicht beabsichtigen, das Tutorial **Intelligenter Raum** zu erkunden, können Sie diese Ressourcen zerstören:

1. Gehen Sie zur AWS IoT Core-Konsole, wählen Sie Act, wählen Sie Regeln, suchen Sie Ihre Regel in der Liste und löschen Sie sie.
2. Gehen Sie zur AWS IoT Events Konsole, wählen Sie Detector-Modelle, suchen Sie Ihr Modell in der Liste und löschen Sie es. Wählen Sie Eingaben, suchen Sie Ihre Eingabe in der Liste und löschen Sie sie.
3. Gehen Sie zur IAM-Konsole, wählen Sie Rollen, suchen Sie in der Liste die Rollen, die Sie für Ihre IoT-Core-Regeln und das IoT-Ereignisdetektormodell festgelegt haben, und löschen Sie sie.
4. Führen Sie den Befehl im [PlatformIO CLI Terminalfenster](../blinky-hello-world/prerequisites.html#öffnen-sie-das-platformio-cli-terminal-fenster) aus dem Ordner **Smart-Thermostat**, um die Gerätefirmware zu löschen und zu verhindern, dass das Gerät angeschlossen wird und Nachrichten an AWS IoT Core sendet. Sie können das Gerät auch ausschalten, indem Sie den Netzschalter 6 Sekunden lang gedrückt halten:
```bash
pio run --environment core2foraws --target erase
```

Das nächste Tutorial, das abgeschlossen werden muss, ist [**Intelligenter Raum**](/de/smart-spaces.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}