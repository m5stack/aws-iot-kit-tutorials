+++
title = "Gerätebereitstellung"
weight = 20
pre = "<b>b. </b>"
+++

In diesem Kapitel stellen Sie Ihr Gerät für die Verbindung mit AWS IoT Core mithilfe des integrierten Microchip ATTECC608 Trust&GO secure element bereit, um eine [TLS-Verbindung](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html) herzustellen. Die integrierte Hardware-"Root of Trust" ermöglicht Ihnen eine vereinfachte und beschleunigte Provision, ohne den privaten Schlüssel preiszugeben. Sie können das Gerätezertifikat ([öffentlicher Schlüssel](https://en.wikipedia.org/wiki/Public-key_cryptography)) abrufen, das in das Gerät integriert ist, um ein AWS IoT-Objekt (eine Darstellung Ihres Geräts) zu erstellen. Die eindeutige Seriennummer des sicheren Elements wird als Client-ID verwendet, um das Gerät in AWS IoT Core zu registrieren und zu identifizieren. Sie können ähnliche Prozesse verwenden, um die Bereitstellung einer Flotte mit Tausenden oder Millionen von Geräten gleichzeitig zu automatisieren.

## Blink Hello World Projekt öffnen
Wenn Sie bereits ein anderes Projekt in VS-Code geöffnet haben, öffnen Sie zuerst ein neues Fenster (**Datei** → **Neues Fenster**), um einen leeren Datei-Explorer und die Arbeitsumgebung zu erhalten.

Für dieses Tutorial verwenden Sie das Projekt Blinky-Hello-World. Klicken Sie in Ihrem neuen VS-Code-Fenster auf das **PlatformIO-Logo** in der VS-Code-Aktivitätsleiste (ganz links im Menü), wählen Sie im linken PlatformIO-Menü **Öffnen** aus, klicken Sie auf **Projekt öffnen**, navigieren Sie zum Ordner `Core2-for-aws-IoT-Edukit/Blinky-Hello-World` und klicken Sie auf **Öffnen**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - PIO Menü öffnen, 2 - PIO Home öffnen, 3 - Projekt öffnen" >}}

## Gerätezertifikat abrufen und AWS IoT-Ding registrieren
Um eine sichere TLS-Verbindung über [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) zu AWS IoT Core herzustellen, müssen Sie ein Objekt registrieren, [das Gerätezertifikat anhängen](https://docs.aws.amazon.com/iot/latest/developerguide/register-device-cert.html) (öffentlicher Schlüssel) und die [Richtlinie](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) definieren und dem Zertifikat anhängen. So stellen Sie sicher, dass nicht autorisierte Geräte oder nicht autorisierte Vorgänge nicht in Ihrem AWS-Konto ausgeführt werden. Durch die Aufnahme eines sicheren Elements auf der Core2 des AWS IoT EduKit können wir den Registrierungsprozess automatisieren, ohne jemals sensible private Schlüssel offenzulegen oder zu verwalten. Im Projekt ist ein Script enthalten, das den Prozess automatisiert. Das Skript ruft das vorab bereitgestellte Gerätezertifikat vom sicheren Element der Referenzhardware ab, legt das Gerätezertifikat und zusätzliche Gerätemadaten in einer Manifestdatei ab und signiert die Manifestdatei mit einem lokal generierten [X.509-Zertifikat](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html#x509-client-cert-basics). Das Skript verwendet dann die [Microchip TrustPlatform Tools](https://github.com/MicrochipTech/cryptoauth_trustplatform_designsuite), um die Manifestdatei von der Festplatte zu lesen, zu überprüfen, ob der Inhalt nicht manipuliert wurde. Dann führt sie eine [Just-in-Time-Registrierung](https://aws.amazon.com/blogs/iot/just-in-time-registration-of-device-certificates-on-aws-iot/) durch. Die Microchip (Secure Element Manufacturer) Certificate Authority (CA) registriert das Object in AWS IoT mit dem Gerätezertifikat, fügt dem AWS IoT-Objekt eine die entsprechende Richtlinie hinzu. Dann fügt sie die Endpunktadresse des AWS IoT MQTT-Broker zur Geräte-Firmware-Konfiguration hinzu. Dieser Prozess kann so skaliert werden, dass Zehntausende von Geräten gleichzeitig in AWS IoT integriert werden.

So führen Sie das Registrierungs-Skript mit dem [PlatformIO CLI Terminal-Fenster](prerequisites.html#öffnen-sie-das-platformio-cli-terminal-fenster) aus:

```bash
cd Blinky-Hello-World
pio run -e core2foraws-device_reg -t register_thing
```

## Abschluss des Kapitels
In diesem Kapitel haben Sie das sichere Element und das Registrierungsskript verwendet, um ein AWS IoT [Objekt](https://docs.aws.amazon.com/iot/latest/developerguide/thing-registry.html) zu erstellen, eine [Richtlinie](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html) für Ihr Objekt zu definieren und diese an das Gerätezertifikat anzuhängen. All dies geschah, ohne jemals den geheimen privaten Schlüssel preiszugeben, und auf diese Weise sind mir einer möglichen Gefährdung entgangen.

Weiter zu [**Mit AWS IoT Core verbinden**](connecting-to-aws.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}