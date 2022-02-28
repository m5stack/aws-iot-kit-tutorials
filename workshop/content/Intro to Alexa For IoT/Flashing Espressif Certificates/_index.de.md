+++
title = "Flashen der Espressif Zertifikate"
weight = 20
pre = "<b>b. </b>"
+++

## Einführung in das Kapitel
Am Ende dieses Kapitels haben Sie die zugehörigen Telefonanwendungen auf Ihrem Mobilgerät installiert, das erforderliche Code-Repository geklont, Zertifikate für die Verbindung mit AWS IoT über das AWS-Konto von Espressif Alexa erhalten und die Zertifikate auf separate Flash-Partitionen auf der Referenzhardware geflasht. .

## Mobile Begleitanwendungen
Um die Authentifizierung mit Alexa abzuschließen, verwenden Sie die Begleit-App von Espressif, um das WLAN des Geräts bereitzustellen und die Referenzhardware mit Ihrem Alexa-Konto bereitzustellen.

Laden Sie die ESP Alexa Telefon-App herunter:
[iOS] (https://apps.apple.com/in/app/esp-alexa/id1464127534)/[Android] (https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

Wir empfehlen (optional_ für dieses Tutorial), dass Sie die Amazon Alexa-App für [iOS] (https://apps.apple.com/us/app/amazon-alexa/id944011620) oder [Android] (https://play.google.com/store/apps/details?id=com.amazon.dee.app) verfügbar haben - dies ist dieselbe App, die für die meisten Alexa-fähigen Geräte verwendet wird.

## Zugriff auf den Code
Der gesamte Code für dieses Tutorial befindet sich im Ordner `Alexa_for_IoT-Intro` aus dem Repository, das Sie im Tutorial [**Blinky Hello World**] (/en/blinky-hello-world.html) geklont haben. Wenn Sie das Repository bereits lokal geklont haben, können Sie diesen Abschnitt überspringen.

Um das Repo erneut aus dem [PlatformIO CLI Terminal-Fenster] zu klonen (.. /blinky-hello-world/prerequisites.html #open -das-Plattform-Cli-Terminal-Fenster):
```
git-Klon https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```

## Öffnen der Projektumgebung
Für dieses Tutorial verwenden Sie das Projekt Alexa_for_IoT-Intro. In Ihrem neuen VS-Code-Fenster
1. Klicken Sie in der VS-Code-Aktivitätsleiste auf das **PlatformIo-Logo** (Menü ganz links)
2. Wählen Sie im linken PlatformIO-Menü **Öffnen**
3. Klicken Sie auf**Projekt öffnen**
4. Navigieren Sie zum Ordner `Core2-for-aws-iot-edukit/alexa_for_IOT-Intro` und klicken Sie auf **Öffnen**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Open PIO menu, 2 - Open PIO home, 3 - Open the project folder" >}}


Als Nächstes müssen Sie ein neues PlatformIO CLI-Terminalfenster in VS-Code öffnen:
1) Klicken Sie auf das **PlatformIo-Logo** in der VS-Code-Aktivitätsleiste (Menü ganz links).
2) Wählen Sie im Menü **Schnellzugriff** unter **Verschiedenes** die Option **Neues Terminal**.

{{< img "pio-new_terminal-smart_thermostat.en.png" "PlatformIO CLI terminal in VS Code" "1 - Open PIO menu, 2 - Open new PIO Terminal, 3 - Verify you're in the 'PlatformIO CLI' terminal session, 4 - Paste the commands into terminal, 5 - If you encounter an error autodetecting the port, open the Platform.ini file and follow instructions to manually add the serial port.">}}

## Richten Sie AWS IoT-Zertifikate ein
Sie müssen die AWS IoT-Anmeldeinformationen erstellen, um mit dem AWS IoT-Kern zu kommunizieren. Für diesen Workshop und dieses Gerät hat Espressif AWS IoT-Anmeldeinformationen bereitgestellt, die in ihrem AWS-Konto mit der M5Stack Core2 for AWS IoT EduKit-Referenzhardware verwendet werden können. Um die Anmeldeinformationen für Ihr Gerät zu erhalten, um eine Verbindung zu seinem Dienst herzustellen, füllen Sie das Formular [hier] aus (https://espressif.github.io/esp-va-sdk/).

Nachdem Sie die E-Mail mit der ZIP-Datei für Anmeldeinformationen erhalten haben, speichern Sie die Datei und entpacken Sie den Inhalt. Nach dem Extrahieren haben Sie einen Ordner mit dem Namen **espcredentials**. Wenn das Gerät angeschlossen ist, können Sie diese Zertifikate auf Ihr Gerät hochladen, indem Sie die folgenden Befehle in Ihrem PlatformIO CLI-Terminalfenster eingeben:

{{%expand "Ubuntu or macOS" %}}
1. Kopieren Sie den Inhalt von espcredentials in den Ordner `core2-for-aws-iot-edukit/alexa_for_iot-intro/ESP_ALEXA_CREDENTIALS` *mit Ausnahme der Datei**mfg_config.csv**. Ersetzen Sie **< <PATH_TO>>** durch den Speicherort des Ordner**espcredentials**. Das nachgestellte **/** der Quelle weist [rsync] (https://download.samba.org/pub/rsync/rsync.1) an, den Inhalt in das Verzeichnis `zu kopieren. /esp_alexa_credentials/ `Ziel. Wenn Sie es ausschließen, wird der Ordner kopiert und im nächsten Schritt wird die Datei nicht gefunden:
   ```bash
   rsync -avr --exclude='mfg_config.csv' <<PATH_TO>>/espcredentials/ ./esp_alexa_credentials/
   ```
2. Verwenden Sie den folgenden PIO-Befehl, um ein Skript auszuführen, das eine Binärdatei mit den Anmeldeinformationsdateien namens „mfg.bin“ im Ordner esp_alexa_credentials erstellt und dann die Binärdatei auf eine Partition des integrierten [nicht-flüchtigen Flash-Speichers] hochlädt ( https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/storage/nvs_flash.html) (NVS), auf die später von der Anwendung als Schlüssel/Wert-Paar zugegriffen wird:
   ```bash
   pio run --environment core2foraws --target flash_certificates
   ```
{{% /expand%}}
{{%expand "Windows" %}}
1. Kopieren Sie den Inhalt von espcredentials in den Ordner `core2-for-aws-iot-edukit/alexa_for_iot-intro/ESP_ALEXA_CREDENTIALS` *mit Ausnahme der Datei**mfg_config.csv**. Ersetzen Sie **< <PATH_TO>>** durch den Speicherort des Ordner**espcredentials**. Das nachgestellte **/** der Quelle weist [rsync] (https://download.samba.org/pub/rsync/rsync.1) an, den Inhalt in das Verzeichnis `zu kopieren. /esp_alexa_credentials/ `Ziel. Wenn Sie es ausschließen, wird der Ordner kopiert und im nächsten Schritt wird die Datei nicht gefunden:
   ```PowerShell
   robocopy "<<PATH_TO>>\espcredentials\" ".\esp_alexa_credentials\" /xf mfg_config.csv
   ```
2. Verwenden Sie den folgenden PIO-Befehl, um ein Skript auszuführen, das eine Binärdatei mit den Anmeldeinformationsdateien namens „mfg.bin“ im Ordner esp_alexa_credentials erstellt und dann die Binärdatei auf eine Partition des integrierten [nicht-flüchtigen Flash-Speichers] hochlädt ( https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/storage/nvs_flash.html) (NVS), auf die später von der Anwendung als Schlüssel/Wert-Paar zugegriffen wird:
   ```PowerShell
   pio run --environment core2foraws --target flash_certificates
   ```
{{% /expand%}}


Wenn alles eingerichtet und fertig ist, gehen wir zu [**AFI** bauen und testen] (/en/intro-to-alexa-for-iot/building-and-testing-afi.html) über.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}