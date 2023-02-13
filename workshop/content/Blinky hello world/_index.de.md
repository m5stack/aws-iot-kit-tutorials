+++
title = "Blinky Hello World"
chapter = true
weight = 30
pre = "<b>2. </b>"
+++

Erfahren Sie, wie Sie mit Ihrem M5Stack Core2 des AWS IoT Kit eine mit der Cloud verbundene „blinkende“-Anwendung (das [Hallo Welt Programm](https://de.wikipedia.org/wiki/Hallo-Welt-Programm) für Mikrocontroller) erstellen. In Ihrem eigenen AWS-Konto lernen Sie, wie Sie Ihr Gerät mit [AWS IoT Core](https://aws.amazon.com/iot-core/) verbinden, [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html)-Nachrichten senden und empfangen und die Nachrichten sowohl auf dem Gerät als auch in der AWS-Cloud anzeigen. Alle hier verwendeten Schritte und Fähigkeiten bilden die Grundlage für den Erfolg in den nachfolgenden Tutorials. Insbesondere werden Sie:
- Die [AWS CLI](https://aws.amazon.com/cli/) auf Ihrem Host-Computer installieren und konfigurieren, um Ihre AWS-Services remote zu verwalten und bereitgestellte Hilfsskripte zu verwenden.
- Registrieren Sie ein „[Objekt](https://docs.aws.amazon.com/iot/latest/developerguide/iot-thing-management.html)“ in AWS IoT Core mithilfe der Zertifikate, die zuvor auf dem integrierten sicheren Element bereitgestellt wurden.
- Konfigurieren Sie Ihre Referenzhardware für die Verbindung mit Wi-Fi, verbinden Sie sich mit dem [AWS IoT-Endpunkt](https://docs.aws.amazon.com/general/latest/gr/iot-core.html) Ihres Kontos und senden Sie MQTT-Nachrichten an AWS IoT Core.
- Empfangen Sie eine MQTT-Nachricht von der Cloud auf der verwendeten Referenzhardware, um das Blinken einer LED auszulösen.

Der gesamte Inhalt dieses Tutorials setzt voraus, dass Sie derzeit das [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT Kit](https://www.amazon.com/dp/B08VGRZYJR/) in Ihrem Besitz haben, über ein [AWS-Konto](https://console.aws.amazon.com/console/home) verfügen, das *nicht* Produktionsworkloads ausführt. Zudem sollten Sie die [Voraussetzungen](getting-started/prerequisites.html) aus dem Tutorial **Erste Schritte** durchgeführt haben und mit grundlegenden technischen Konzepten und Tools wie der Eingabeaufforderung/dem Terminal vertraut. Um Ihr eigenes Kit zu kaufen, besuchen Sie [Amazon.com](https://www.amazon.com/dp/B08VGRZYJR/), den [M5Stack-Shop](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) oder deren weltweite Vertriebspartner. Wenn Sie sich noch nicht für ein AWS-Konto registriert haben, [erstellen Sie ein Konto](https://portal.aws.amazon.com/billing/signup).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}