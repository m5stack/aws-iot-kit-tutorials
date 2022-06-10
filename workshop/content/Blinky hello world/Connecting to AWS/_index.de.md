+++
title = "Verbinden mit AWS IoT Core"
weight = 30
pre = "<b>c. </b>"
+++

In diesem Kapitel werden Sie die Firmware Ihres Geräts konfigurieren, erstellen und flashen, sodass Ihr Gerät eine Verbindung zu Ihrem Wi-Fi-Netzwerk und AWS IoT Core herstellen kann. Um eine Verbindung zu AWS IoT Core herzustellen und damit zu kommunizieren, müssen Sie Ihr Gerät mit Wi-Fi-Anmeldeinformationen und der URL Ihres [AWS IoT-Endpunkts](https://docs.aws.amazon.com/iot/latest/developerguide/connect-to-iot.html#iot-device-endpoint-intro) konfigurieren. Der Aufbau einer sicheren MQTT-Verbindung wird mit dem [AWS IoT Device SDK in Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) und den vorab bereitgestellten sicheren Hardwarezertifikaten im integrierten [Microchip ATECC608 Trust&Go](https://www.microchip.com/wwwproducts/en/ATECC608B-TNGTLS) durchgeführt. Mit dem sicheren Element müssen Sie keine Zertifikate von AWS IoT Core abrufen oder eigene generieren, um eine Verbindung herzustellen. Die Konnektivitätsbibliotheken im AWS IoT Device SDK für Embedded C vereinfachen die Konnektivität und den Zugriff auf AWS IoT-Services und -Funktionen.

## Konfiguration der ESP32-Firmware
Die Konfiguration Ihres Quellcodes erfolgt über [Kconfig](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html). Kconfig ist dasselbe Konfigurationssystem, das vom Linux-Kernel verwendet wird, und hilft dabei, die verfügbaren Konfigurationsoptionen (Symbole) in einer Baumstruktur zu vereinfachen.

Jetzt gehen Sie in das KConfig-Menü, um die erforderlichen [Symbole](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html) zu konfigurieren, damit das Gerät eine Verbindung zu Ihrem 2,4-GHz-Wi-Fi-Netzwerk herstellen kann. Wechseln Sie zunächst in das Verzeichnis **Blinky-Hello-World** des Repositorys im PIO-Terminalfenster und geben Sie ein:
```bash
pio run —environment core2foraws —target menuconfig
```

{{< img "idf_menuconfig-wifi.en.webp" "Configuring Core2 for AWS IoT EduKit with p.py menuconfig" >}}

Verwenden Sie die Richtungstasten (oder *K* und *J* oder *-* und *+*) auf Ihrer Tastatur, um im Menü zu **AWS IoT EduKit Configuration** zu gehen. Geben Sie Ihre **WiFi-SSID** und Ihr **WiFi-Passwort** mit Ihren Wi-Fi-Anmeldeinformationen ein. Wenn Sie fertig sind, drücken Sie zum Speichern die Taste *s* auf Ihrer Tastatur. Bestätigen Sie den Speicherort der Datei, indem Sie *Enter* drücken, gefolgt von *q* zum Beenden.

{{% notice warning %}}
Stellen Sie sicher, dass Ihre SSID für ein 2,4-GHz-Netzwerk bestimmt ist. Der ESP32-D0WD auf der M5Stack Core2 für AWS-Hardware unterstützt keine 5-GHz-Wi-Fi-Bänder.
{{% /notice %}}

## Erstellen, Hochladen und Überwachen der Blinky Hello World Firmware
Sie können jetzt die Blinky Hello World Firmware erstellen (kompilieren) und hochladen. Der Vorgang ist derselbe wie beim Tutorial **Erste Schritte** zum Erstellen, Flashen und Überwachen der seriellen Ausgabe:

1) Um die Firmware zu erstellen, fügen Sie den folgenden Befehl ein (es dauert einige Minuten):
    ```bash
    pio run --environment core2foraws
    ```
2) Nachdem das Bauen erfolgreich abgeschlossen ist, wird nun die kompilierte Firmware über USB auf das angeschlossene Gerät hochgeladen. Nutzen Sie die folgende Anweisung die Nachrichten, die über den seriellen Ausgang an den Host-Rechner gesendeten wurden, zu überwachen:
    ```bash
    pio run --environment core2foraws --target upload --target monitor
    ```
{{% notice warning %}}
Wenn Sie während des Uploads oder bei der Überwachung der seriellen Ausgabe eine Fehlermeldung über einen falschen Port oder Timeout erhalten, öffnen Sie die Platformio.ini-Datei und befolgen Sie die dortigen Anweisungen, um den Upload-Port manuell einzustellen.
{{% /notice %}}

## Abschluss des Kapitels
In diesem Kapitel haben Sie Ihr Gerät erfolgreich kompiliert und geflasht und überwachen aktiv seine seriellen Ausgänge. Unter Verwendung des AWS IoT Device SDK für Embedded C wird das AWS IoT Edukit mit dem [MQTT-Nachrichtenbroker](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) (AWS IoT Core) authentifiziert und ist bereit, Nachrichten zu empfangen.

Sie können jetzt zum letzten Kapitel in diesem Tutorial gehen, [**Blinken der LEDs**](/de/blinky-hello-world/blinking-the-leds.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}