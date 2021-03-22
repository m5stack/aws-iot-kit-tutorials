+++
title = "Blinky Hello World"
chapter = true
weight = 20
pre = "<b>2. </b>"
+++

Erfahren Sie, wie Sie eine &quot;Blinky&quot;-Anwendung (die Hallo-Welt für Mikrocontroller) mit Ihrer M5Stack Core2 for AWS IoT EduKit-Hardware erstellen. Unter Verwendung Ihres eigenen AWS-Kontos werden Sie durchgehen, wie Sie sich mit AWS IoT Core verbinden, Nachrichten senden und empfangen und die Nachrichten sowohl auf dem Gerät als auch in der AWS-Cloud anzeigen. Alle Schritte und Fähigkeiten, die hier verwendet werden, bilden die Grundlage, um in den nachfolgenden Tutorials erfolgreich zu sein:

- Installieren und konfigurieren Sie zusätzlich benötigte Software.
- Registrieren Sie ein &quot;Thing&quot; in AWS IoT Core mithilfe der Sicherheitszertifikate, die auf dem Onboard-Secure-Element vorab bereitgestellt wurden.
- Verbinden und Senden von MQTT-Nachrichten von der Referenzhardware zu AWS IoT Core unter Verwendung des ESP-IDF (FreeRTOS-Kernel mit symmetrischer Multiprozessorverarbeitung).
- Empfangen Sie eine MQTT-Nachricht auf der Referenz, um das Blinken einer LED auszulösen.

Der gesamte Inhalt dieses Tutorials setzt voraus, dass Sie das [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT EduKit](https://www.amazon.com/dp/B08NP5LVFH) besitzen, ein [AWS-Konto](https://console.aws.amazon.com/) haben, auf dem keine Produktions-Workloads ausgeführt werden, und mit grundlegenden technischen Konzepten und Tools, wie der Eingabeaufforderung/dem Terminal, vertraut sind. Um Ihr eigenes Kit zu erwerben, schauen Sie bei [Amazon.com](https://www.amazon.com/dp/B08NP5LVFH), im [M5Stack-Shop](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) oder bei den weltweiten Distributoren vorbei. Wenn Sie sich noch kein AWS-Konto besitzen, [erstellen Sie](https://portal.aws.amazon.com/billing/signup) bitte ein Konto.

Um mit diesem Teil zu beginnen, gehen Sie zum ersten Kapitel, [Voraussetzungen](/de/blinky-hello-world/prerequisites.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}