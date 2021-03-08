+++
title = "Device Provisioning"
weight = 20
pre = "<b>b. </b>"
+++

In diesem Kapitel stellen Sie das Gerät für die Konnektivität mit AWS IoT Core bereit, indem Sie das integrierte sichere Element ATTECC608 Trust&amp;GO von Microchip verwenden, um eine TLS-Verbindung herzustellen. Die eingebaute Hardware-Vertrauenswurzel ermöglicht Ihnen einen vereinfachten und beschleunigten Bereitstellungspfad, wobei der private Schlüssel nie offengelegt wird. Sie können das im Gerät eingebaute Gerätezertifikat abrufen und eine Manifestdatei erstellen, um ein AWS IoT-Thing (eine Darstellung und Aufzeichnung Ihres Geräts) zu erstellen. Die Client-ID dieses Geräts wird in AWS IoT Core anhand der Seriennummer des sicheren Elements registriert und identifiziert. Sie können ähnliche Prozesse verwenden, um die Flottenbereitstellung von Tausenden oder Millionen von Geräten auf einmal zu automatisieren.

## Identifizieren des seriellen Anschlusses auf dem Host-Rechner
Bitte beziehen Sie sich auf die [offizielle Dokumentation von Espressif](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/establish-serial-connection.html) zum Herstellen serieller Verbindungen mit dem ESP32. Der Port Ihres Geräts hängt von Ihrem Betriebssystem ab. Unter macOS befindet sich das Gerät typischerweise auf `/dev/cu.SLAB_USBtoUART`. Für Linux liegt das Gerät typischerweise auf `/dev/ttyUSB0` (der Benutzer muss zur Dialout-Gruppe hinzugefügt werden). Unter Windows beginnt es mit einem `COM` und endet mit einer Zahl.

## Abrufen des Gerätezertifikats und Registrieren des AWS IoT-Thing
Wir haben den Prozess des Abrufs des Gerätezertifikats aus dem sicheren Element der Core2 for AWS IoT EduKit-Referenzhardware, der Generierung eines Gerätemanifests durch Signieren des Gerätezertifikats mit einem x.509-Zertifikat (einschließlich Ihres AWS IoT-Registrierungscodes), der Registrierung des Geräts in AWS IoT mit dem Gerätezertifikat und des Anhängens einer sicheren Richtlinie an das AWS IoT-Thing vereinfacht.

Gehen Sie in das Verzeichnis der AWS IoT-Registrierungshilfe des Projekts und installieren Sie die erforderlichen Abhängigkeiten mit pip:
```bash
cd Core2-for-AWS-IoT-EduKit/Blinky-Hello-World/utilities/AWS_IoT_registration_helper/
pip3 install -r requirements.txt
```

Als nächstes müssen Sie das Python-Skript ausführen, das alle Schritte zur Registrierung des Geräts in Ihrem AWS-Konto ausführt. Stellen Sie sicher, dass Sie zuerst **<<DEVICE_PORT>>** durch den seriellen Port ersetzen, an den Ihr Core2 for AWS IoT EduKit-Gerät angeschlossen ist:
```bash
python registration_helper.py -p <<DEVICE_PORT>>
```

{{% notice info %}}
Wenn Sie Ihre Shell schließen oder eine neue Shell öffnen, müssen Sie erneut `conda activate edukit` eingeben, um die virtuelle Umgebung zu reaktivieren und die `export.sh` (macOS/Linux) bzw. `export.bat` (Windows) von ESP-IDF ausführen, um die ESP-IDF-Tools jedes Mal neu zu Ihrem Pfad hinzuzufügen.
{{% /notice %}}

Wenn das Gerät erfolgreich in AWS IoT registriert und bereitgestellt wurde, gehen Sie zurück zum **Blinky-Hello-World** Verzeichnis, indem Sie eingeben:
```bash
cd ../..
```

## Fazit
In diesem Kapitel haben Sie das sichere Element verwendet, um ein AWS [IoT-Thing](https://docs.aws.amazon.com/iot/latest/developerguide/thing-registry.html) zu erstellen, eine [Berechtigungsrichtlinie](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html) für Ihr Thing einzurichten und das Gerätezertifikat damit zu verbinden. All dies wurde getan, ohne den geheimen privaten Schlüssel jemals offenzulegen und eine potenzielle Möglichkeit für dessen Kompromittierung zu bieten.

Weiter zu [Verbinden mit AWS IoT Core](/de/blinky-hello-world/connecting-to-aws.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}