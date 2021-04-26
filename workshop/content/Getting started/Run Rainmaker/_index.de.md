+++
title = "Ausführen des ESP RainMaker-Agenten"
weight = 12
pre = "<b>b. </b>"
+++

Wir sind bereit, unsere Anwendung zu kompilieren und auf das Gerät zu flashen, das Wi-Fi über die ESP Rainmaker Phone App bereitzustellen, einen Benutzer zu registrieren, dem das Gerät zugewiesen wird, und mit der Steuerung unserer Peripheriegeräte über AWS IoT zu beginnen.

## Öffnen des Projekts in PlatformIO

Im Stammverzeichnis des Projekts, das Sie geklont oder heruntergeladen und extrahiert haben, befinden sich mehrere Ordner. Für dieses Tutorial werden wir das Projekt &quot;Getting-Started&quot; in PlatformIO öffnen. Öffnen Sie zunächst Visual Studio Code und nachdem Sie ein paar Sekunden gewartet haben, bis die PlatformIO-Erweiterung geladen ist, klicken Sie auf das **PlatformIO logo** in der linken Leiste, wählen Sie im linken PlatformIO-Menü die Option Open, klicken Sie auf **Open Project**, navigieren Sie zum Ordner Core2-for-AWS-IoT-EduKit/Getting-Started und klicken Sie auf **open**.
{{< img "pio-home.en.png" "PlatformIO home screen" >}}

## Identifizieren und Einstellen des seriellen Anschlusses auf dem Host-Rechner

Sie müssen wissen, welchen Anschluss Sie für die Kommunikation mit dem Gerät zum Hochladen von Firmware oder zum Überwachen der seriellen Ausgabe verwenden werden. Unter macOS liegt das Gerät typischerweise auf **/dev/cu.SLAB_USBtoUART** und die mitgelieferte Konfiguration sollte keine Änderungen erfordern. Unter Linux liegt es typischerweise auf **/dev/ttyUSB0** und erfordert, dass der Benutzer zur Dialout-Gruppe hinzugefügt wird. Unter Windows müssen Sie den Wert durch **COM** ersetzen und mit der entsprechenden Portnummer enden (z. B. COM3). Für spezifischere Anweisungen für Windows oder Linux-Distributionen, die eine zusätzliche Konfiguration erfordern, lesen Sie bitte [die offizielle Dokumentation von Espressif](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/establish-serial-connection.html) zur Einrichtung serieller Verbindungen mit dem ESP32.

Wählen Sie den Dateiexplorer von Visual Studio Code (<i class="far fa-copy"></i> in der linken Leiste), öffnen Sie die Datei **platformio.ini** und ändern Sie den Wert **upload_port** so, dass er dem seriellen Anschluss entspricht, an den das Gerät angeschlossen ist.

{{% notice note %}}
Sehen Sie sich die offiziellen PlatformIO-Dokumente über die Option [upload\_port](https://docs.platformio.org/en/latest/projectconf/section_env_upload.html#upload-port) an, um Formatierungs- und andere Beispiele zu sehen.
{{% /notice %}}

## Erstellen und Hochladen der RainMaker Agent-Firmware

Sie sind nun bereit, Ihre Firmware zu erstellen und zu flashen. Beginnen Sie mit einem Klick auf das PlatformIO-Logo in der linken Leiste. Klicken Sie dann auf das Symbol (fahren Sie mit der Maus neben **PROJECT TASKS**), um die verfügbaren Optionen zu aktualisieren. Klicken Sie auf die Schaltfläche **Build**, um mit der Erstellung der Gerätefirmware zu beginnen. Dies wird je nach Ihrem Computer einige Minuten dauern. Sobald dies abgeschlossen ist, klicken Sie auf die Option **Upload and Monitor** aus dem Menü PlatformIO project tasks. Sobald der Upload erfolgreich abgeschlossen ist, bootet das Gerät mit der Firmware, die gerade kompiliert und hochgeladen wurde. Außerdem wird die serielle Ausgabe des Geräts in diesem Terminal-Viewport (jetzt das serielle Monitorfenster) angezeigt. Das Gerät durchläuft nun den Prozess der Generierung von Sicherheitsschlüsseln und führt einen Selbstanspruch durch. Die Schlüsselgenerierung kann ein paar Sekunden bis zu ein paar Minuten dauern, aber sobald die Beanspruchung abgeschlossen ist, wird ein QR-Code im seriellen Monitorfenster angezeigt.
{{< img "pio-menu.en.png" "PlatformIO Menu to build, flash, monitor" >}}

{{% notice note %}}
Weitere Informationen zur Fehlerbehebung oder FAQs finden Sie in den offiziellen [Espressif RainMaker FAQs](https://rainmaker.espressif.com/docs/faqs.html).
{{% /notice %}}

## Beanspruchen und Bereitstellen des Geräts
Öffnen Sie auf Ihrem Mobiltelefon die ESP RainMaker Phone App, gewähren Sie die angeforderten Berechtigungen für die mobile App, drücken Sie auf &quot;Add Device&quot; (**Gerät hinzufügen**) und scannen Sie dann den QR-Code, der auf dem Serienmonitor Ihres Computers angezeigt wird (_nicht_ den QR-Code auf dem Gerät). Das Gerät durchläuft dann den Bereitstellungsprozess, der die Wi-Fi-Bereitstellung mit Ihren Wi-Fi-Anmeldeinformationen für Ihr 2,4-GHz-Wireless-Heimnetzwerk beinhaltet. Nach einer erfolgreichen Wi-Fi-Verbindung authentifiziert sich das Gerät und Ihre Telefon-App wird mit mehreren virtuellen Geräten aufgefüllt, die angezeigt und/oder gesteuert werden können. Wenn die virtuellen Geräte in der Telefon-App nach ein oder zwei Minuten als **offline** markiert sind, versuchen Sie, zum Aktualisieren nach unten zu scrollen.
{{< img "pio-qr_code_scan.en.png" "Scan the QR code in serial output" >}}

Mit dem virtuellen Gerät, das in der mobilen App aufgelistet und online ist, können Sie den On-Board-Motor oder die LEDs ein- oder ausschalten, die Geschwindigkeit des Motors anpassen, die Farbe und Helligkeit der LED-Balken einstellen und die interne Gerätetemperatur anzeigen.

{{% notice info %}}
Wenn Sie die falschen Wi-Fi-Anmeldeinformationen eingegeben haben, müssen Sie zunächst die [Firmware löschen](/de/getting-started/run-rainmaker.html#löschen-der-firmware-mit-platformio), die Espressif RainMaker Agent-Firmware erneut auf das Gerät laden und das Gerät erneut mit Ihrem Mobiltelefon hinzufügen.
{{% /notice %}}

## Löschen der Firmware mit PlatformIO
Wenn Sie mit dieser Anwendung fertig sind und zu anderen Tutorials übergehen möchten, müssen Sie zuerst die Gerätefirmware löschen. Sie müssen jedoch zuerst den aktiven seriellen Monitor stoppen, da er die weitere Kommunikation mit dem Anschluss blockiert. Sie können den seriellen Monitor stoppen, indem Sie das Visual Studio Code-Fenster bei laufendem seriellen Monitor auswählen und die Tastenkombination **CTRL** + **C** drücken. Um den Flash-Speicher zu löschen, öffnen Sie das Menü PlatformIO, erweitern Sie die Menüliste **Platform** und klicken Sie dann auf die Option **Erase Flash**. Dadurch werden alle Daten auf dem Gerät gelöscht, einschließlich der verwendeten RainMaker-Zertifikate. 
{{< img "pio-qr_code_scan.en.png" "Erasing flash with PlatformIO" >}}

{{% notice note %}}
Wenn Sie das Gerät mit einem leeren Flash ausschalten, ist der Bildschirm des Geräts leer und ein Ticken ertönt aus dem Lautsprecher. Dies ist ein erwartetes Verhalten, da sich das Gerät ständig neu startet, wenn keine Anwendung ausgeführt wird.
{{% /notice %}}

## Fazit
Sie haben gerade eine Connected Home-Anwendung über das AWS IoT EduKit-Programm erstellt! Nicht nur das, Sie haben auch die notwendigen Tools zum Erstellen, Bearbeiten, Kompilieren und Flashen von eingebettetem Code auf Ihrem Gerät! In den nächsten Tutorials werden Sie mehr in die Praxis gehen und die Fähigkeiten erlernen, um mit dem Aufbau Ihrer eigenen End-to-End-IoT-Lösungen zu beginnen.


Weiter zu [**Blinky Hello World**](/de/blinky-hello-world.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}