+++
title = "Flashen der Espressif Zertifikate"
weight = 20
pre = "<b>b. </b>"
+++

## Kapiteleinleitung
Am Ende dieses Kapitels haben Sie die Companion-Anwendungen auf Ihrem Mobilgerät installiert, das erforderliche Code-Repository geklont, Zertifikate für die Verbindung mit AWS IoT über das AWS-Konto von Espressif Alexa erhalten und die Zertifikate auf separate Flash-Partitionen der Referenzhardware geflasht.

## Mobile Companion-Apps
Um die Authentifizierung mit Alexa abzuschließen, benötigen Sie die Companion-App von Espressif, um das Gerät mit WiFi zu versorgen und die Referenzhardware mit Ihrem Alexa-Konto zu versehen.

Laden Sie die ESP Alexa Phone App herunter:
[iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

Es wird empfohlen (_optional_ für dieses Tutorial), die Amazon Alexa-App für [iOS](https://apps.apple.com/us/app/amazon-alexa/id944011620) und [Android](https://play.google.com/store/apps/details?id=com.amazon.dee.app) zur Verfügung zu haben - dies ist die gleiche App, die zur Bereitstellung der meisten Alexa-fähigen Geräte verwendet wird.

## Zugriff auf den Code
Der gesamte Code für dieses Tutorial befindet sich im Ordner `Alexa_for_IoT-Intro` aus dem Repo, das Sie im [**Blinky Hello World**](/de/blinky-hello-world.html) geklont haben. So klonen Sie das Repository erneut:
```
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```

## AWS IoT-Zertifikate einrichten
Sie müssen die AWS IoT-Zertifikate erstellen, um mit AWS IoT Core zu kommunizieren. Für diesen Workshop und dieses Gerät hat Espressif die AWS IoT-Zertifikate bereitgestellt, die mit der Referenzhardware "M5Stack Core2 for AWS IoT EduKit" verwendet werden können. Befolgen Sie die Schritte [hier](https://espressif.github.io/esp-va-sdk/#/), um Ihre Zertifikate zu erhalten.

Nach dem Entpacken der heruntergeladenen Zertifikate haben Sie einen Ordner namens **espcredentials**. Navigieren Sie zu diesem Ordner und ändern Sie die Datei **mfg_config.csv** (die mit den Zertifikaten bereitgestellt wird), um die korrekten Pfade für jede Datei wiederzugeben. Führen Sie dann die folgenden Befehle im selben Ordner aus, um die Zertifikate in das Gerät zu flashen. Ersetzen Sie **<<DEVICE_PORT>>** durch den seriellen Port, an den Ihr Core2 for AWS IoT EduKit-Gerät angeschlossen ist:

```bash
python $IDF_PATH/components/nvs_flash/nvs_partition_generator/nvs_partition_gen.py generate /path/to/mfg_config.csv mfg.bin 0x6000

python $IDF_PATH/components/esptool_py/esptool/esptool.py --chip esp32 --port <<DEVICE_PORT>> write_flash 0x10000 mfg.bin
```

Da nun alles eingerichtet und vorbereitet ist, können wir mit dem [**Build and Testen der AFI**](/de/intro-to-alexa-for-iot/building-and-testing-afi.html) fortfahren.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}