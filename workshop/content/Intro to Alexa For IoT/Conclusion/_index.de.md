
+++
title = "Zusammenfassung"
weight = 50
pre = "<b>e. </b>"
+++

## Zusammenfassung
Sie haben das AWS IoT-EduKit-Praxistutorial für die Ausführung von Alexa für AWS IoT (AFI) unter Verwendung des benutzerdefinierten portierten Espressif Voice Assistant SDK und Alexa zur Steuerung von On-Board-Peripheriegeräten mit Alexa Smart Home-Befehlen abgeschlossen. Weitere Informationen zu Smart Home für AVS finden Sie in der [Dokumentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/smart-home-for-avs.html) oder im folgenden [Blog-Beitrag](https://developer.amazon.com/en-US/blogs/alexa/device-makers/2020/04/create-a-sample-alexa-built-in-disco-ball-with-smart-home-for-av).

Wenn Sie mehr über andere verfügbare Schnittstellen erfahren oder eigene Implementierungen für die Verwendung anderer Funktionen der "M5Stack Core2 for AWS IoT EduKit"-Referenzhardware entwickeln möchten, werfen Sie einen Blick auf [ModeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-modecontroller.html) und [ToggleController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-togglecontroller.html). Nutzen Sie Ihre Kreativität und das, was Sie in den anderen Kapiteln dieses Tutorials gelernt haben, um neue Möglichkeiten für die Sprachsteuerung zu entdecken!

Dies ist das letzte derzeit verfügbare Tutorial. Wir werden weitere veröffentlichen, aber für den Moment haben Sie die Fähigkeiten, um Ihre eigene IoT-Lösung mit AWS-Services mit den bisher erlernten Fähigkeiten zu bauen.

## Aufräumen
In dieser Lösung haben Sie keine Ihrer eigenen Ressourcen in AWS verwendet. Sie können jedoch Ihre Gerätesoftware löschen (und die Zertifikate aus dem Flash entfernen), indem Sie den Befehl eingeben (ersetzen Sie **<<DEVICE_PORT>>** durch den seriellen Port, an den Ihr Gerät angeschlossen ist):
```bash
idf.py erase_flash -p <<DEVICE_PORT>>
```

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}