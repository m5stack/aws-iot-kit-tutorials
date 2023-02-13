
+++
title = "Zusammenfassung"
weight = 50
pre = "<b>e. </b>"
+++

## Fazit
Sie haben das praktische AWS IoT Kit Tutorial zum Ausführen von Alexa für AWS IoT (AFI) mit dem benutzerdefinierten portierten Espressif Voice Assistant SDK und Alexa zur Steuerung von On-Board-Peripheriegeräten mit Alexa Smart Home-Befehlen abgeschlossen. Weitere Informationen zu Smart Home für AVS finden Sie in der [Dokumentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/smart-home-for-avs.html) oder im folgenden [Blogbeitrag](https://developer.amazon.com/en-US/blogs/alexa/device-makers/2020/04/create-a-sample-alexa-built-in-disco-ball-with-smart-home-for-av).

Wenn Sie mehr über andere verfügbare Schnittstellen erfahren oder Ihre eigene Implementierung mit anderen Funktionen der M5Stack Core2 des AWS IoT Kit entwickeln möchten, schauen Sie sich [ModeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-modecontroller.html) und [Toggle Controller](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-togglecontroller.html) an. Nutzen Sie Ihre Kreativität und das, was Sie in den anderen Kapiteln dieses Tutorials gelernt haben, um neue Funktionen mittels Sprache zu entdecken!

Dies ist das letzte derzeit verfügbare Tutorial. Wir werden mehr veröffentlichen, aber im Moment können Sie Ihre eigene IoT-Lösung mit AWS-Services erstellen, indem Sie die bisher erlernten Fähigkeiten nutzen.

## Aufräumen
In dieser Lösung haben Sie keine Ihrer eigenen Ressourcen in AWS verwendet. Wenn Sie jedoch mit der Alexa-Anwendung fertig sind, können Sie den Flash-Speicher Ihres Geräts vollständig löschen (und die Zertifikate aus dem Flash entfernen), indem Sie diesen Befehl im [PIO CLI-Terminalfenster](../blinky-hello-world/prerequisites.html#Öffnen-Sie-das-PlatformIO-CLI-Terminal-Fenster) eingeben:
```bash
pio run --environment core2foraws --target erase
```

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}