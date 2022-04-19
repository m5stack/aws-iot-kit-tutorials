+++
title = "Ausführen des ESP RainMaker-Agenten"
weight = 12
pre = "<b>b. </b>"
+++

Wir sind bereit, unsere Anwendung kompilieren und auf das Gerät hochzuladen, das WLAN über die ESP RainMaker App einzurichten und einen Benutzer zu registrieren, dem die Geräte zugewiesen werden sollen. Die integrierten Peripheriegeräte werden dann als virtuelle Smart-Home-Geräte über AWS IoT gesteuert.

## Das Projekt in PlatformIO öffnen
Im Stammverzeichnis des Repositorys, das Sie im letzten Kapitel von GitHub geklont haben, befinden sich mehrere Ordner. In diesem Tutorial verwenden Sie das Projekt Getting-Started und führen die erforderlichen Vorgänge in PlatformIO aus. Öffnen Sie zuerst Visual Studio Code und nachdem Sie einige Sekunden auf das Laden der PlatformIO-Erweiterung gewartet haben, klicken Sie auf das **PlatformIO-Logo** in VS Code (ganz links im Menü), wählen Sie **Öffnen** aus dem linken PlatformIO-Menü und klicken Sie auf **Projekt öffnen**. Navigieren Sie zum Ordner `Core2-for-AWS-IOT-Edukit/Getting-Started` und klicken Sie auf**öffnen**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Menü öffnen, 2 - PIO Home öffnen, 3 - Getting Started Projekt öffnen" >}}

## Erstellen und Hochladen der RainMaker Agent Firmware
Sie können jetzt die RainMaker Agent-Firmware erstellen (kompilieren) und hochladen. Die Kompilierung erfolgt durch den [GCC-Compiler](https://gcc.gnu.org/onlinedocs/gcc/), der den bereitgestellten lesbaren Code in [Objektcode](https://de.wikipedia.org/wiki/Objektcode) als [elf](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) und Binärdateien (die Firmware) umwandelt. Diese Dateien werden dann in den integrierten Flash des Geräts hochgeladen, damit sie über die virtuelle serielle Schnittstelle ausgeführt werden können. Die serielle Schnittstelle (über UART) ermöglicht eine bidirektionale Kommunikation, sodass Sie Daten vom Gerät zum Host-Computer empfangen können. Um die Firmware zu erstellen und dann hochzuladen und die Ausgabe des Geräts über die serielle Schnittstelle mit PlatformIO zu überwachen:
1) Klicken Sie auf das **PlatformIo-Logo** in der VS-Code-Aktivitätsleiste (Menü ganz links).
2) Wählen Sie im Menü **Quick Access** unter **Miscellaneous** die Option **New Terminal**.
3) Um den Build-Prozess zu starten, fügen Sie den folgenden Befehl (dies dauert einige Minuten) in das neue Terminalfenster ein, das in VS-Code geöffnet wurde:
    ```bash
    pio run --environment core2foraws
    ```

{{% notice info%}}
Im Hintergrund werden von PIO Abhängigkeiten für die Geräteplattform installiert. Wenn diese Vorgänge unvollständig sind, tritt möglicherweise ein Fehler auf. Ein oder zwei Minuten zu warten und den obigen Befehl erneut auszuführen, sollte das Problem beheben.
{{% /notice %}}

{{< img "pio-new_terminal-gs.en.png" "PlatformIO CLI terminal in VS Code" "1 - Menü öffnen, 2 - Neues PIO Terminal öffnen, 3 - Sicherstellen, dass wir eine 'PlatformIO CLI' Terminal Sitzung haben, 4 - Kommando ins Terminal Fenster kopieren, 5 - Bei Problemen, öffnen Sie die Platform.ini Datei und befolgen Sie die Anweisungen zum manuellen Hinzufügen des seriellen Ports.">}}

Jetzt ist es an der Zeit, die kompilierte Firmware über USB auf das angeschlossene Gerät hochzuladen. Nutzen Sie die folgende Anweisung die Nachrichten, die über den seriellen Ausgang an den Host-Rechner gesendeten wurden, zu überwachen.
   ```bash
   pio run --environment core2foraws --target upload --target monitor
   ```
{{% notice info %}}
Wenn Sie während des Uploads oder der Überwachung der seriellen Ausgabe eine Fehlermeldung über falsche Ports oder Timeouts erhalten, öffnen Sie die Platformio.ini Datei und befolgen Sie die Anweisungen in dieser Datei, um den Upload-Port manuell festzulegen.
{{% /notice %}}

## Gerät beanspruchen und bereitstellen
Sobald der Upload erfolgreich abgeschlossen wurde, bootet das Gerät mit der Firmware, die gerade kompiliert und hochgeladen wurde. Sie wird auch in der serielle Ausgabe des Geräts in diesem Terminal-Viewport angezeigt. Das Gerät durchläuft den Prozess der Generierung von Sicherheitsschlüsseln und führt einen sogenannten ["assisted Claim"](https://rainmaker.espressif.com/docs/claiming.html#assisted-claiming-esp32) durch. Die Schlüsselgenerierung kann einige Sekunden bis zu einigen Minuten dauern, aber sobald die Inanspruchnahme abgeschlossen ist, wird ein QR-Code im Terminal-Fenster angezeigt.

Öffnen Sie auf Ihrem Mobiltelefon die ESP RainMaker App, erteilen Sie der angeforderten mobilen App Berechtigungen, drücken Sie **Gerät hinzufügen** und scannen Sie dann die QR-Code-Ausgabe über den seriellen Monitor im Terminal-Viewport. Anschließend wird der Bereitstellungsprozess durchlaufen, der die Wi-Fi-Bereitstellung mit Ihren Wi-Fi-Anmeldeinformationen für Ihr drahtloses 2,4-GHz-Heimnetzwerk umfasst. Nach einer erfolgreichen Wi-Fi-Verbindung authentifiziert sich das Gerät selbst und Ihre Telefon-App wird mit mehreren virtuellen Geräten gefüllt, die angezeigt und/oder gesteuert werden können. Wenn die virtuellen Geräte nach ein oder zwei Minuten in der Telefon-App als „offline**“ gekennzeichnet sind, scrollen Sie zum Aktualisieren nach unten.
{{< img "pio-qr_code_scan.en.png" "Scan the QR code in serial output" >}}

Mit dem virtuellen Gerät, das in Ihrer mobilen App online aufgeführt ist, können Sie den Motor oder die LEDs ein- oder ausschalten, die Geschwindigkeit des Motors anpassen, die Farbe und Helligkeit der LED-Leisten einstellen und die Temperatur des internen Geräts anzeigen.

{{% notice info %}}
Wenn Sie die falschen Wi-Fi-Anmeldeinformationen eingegeben haben, müssen Sie zuerst [die Firmware löschen](/de/getting-started/run-rainmaker.html#löschen-der-firmware-mit-platformio), die Espressif RainMaker Agent Firmware erneut auf das Gerät hochladen und das Gerät erneut mit Ihrem Handy hinzufügen. Weitere Informationen zur Fehlerbehebung oder häufig gestellten Fragen finden Sie in den offiziellen [Espressif RainMaker FAQs](https://rainmaker.espressif.com/docs/faqs.html).
{{% /notice %}}

## Löschen der Firmware mit PlatformIO
Sobald Sie mit dieser Anwendung fertig sind und mit anderen Tutorials fortfahren können, müssen Sie zuerst die Firmware löschen. Sie müssen jedoch zuerst den aktiven seriellen Monitor beenden, da er den virtuellen seriellen Kommunikationsanschluss blockiert. Sie können den seriellen Monitor anhalten, indem Sie **STRG** + **C** drücken. Um die Firmware aus dem Flash-Speicher des Geräts zu löschen, geben Sie den Befehl ein:
```bash
pio run --environment core2foraws --target erase
```

{{% notice info %}}
Wenn Sie das Gerät mit einem leeren Flash-Speicher einschalten, ist der Bildschirm des Geräts schwarz und aus dem Lautsprecher ertönt ein hörbares Ticken. Dies ist ein erwartetes Verhalten, da sich das Gerät dauerhaft selbst neu startet, ohne dass eine Anwendung ausgeführt werden muss.
{{% /notice %}}

## Fazit
Sie haben gerade eine Connected-Home-Anwendung mit dem AWS IoT EduKit-Programm erstellt! Darüber hinaus verfügen Sie über die Tools, die zum Erstellen, Bearbeiten, Kompilieren und Flashen von eingebettetem Code auf Ihrem Gerät erforderlich sind! In den nächsten Tutorials werden Sie mehr praktische Erfahrungen sammeln und lernen die Fähigkeiten, um Ihre eigenen End-to-End-IoT-Lösungen zu entwickeln.

Weiter zu [**Blinky Hello World**](/de/blinky-hello-world.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}