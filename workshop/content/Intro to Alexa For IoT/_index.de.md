+++
title = "Intro zu Alexa für IoT"
chapter = true
weight = 50
pre = "<b>5. </b>"
+++

In diesem Tutorial implementieren Sie die Alexa Voice Service Integration for AWS IoT (AFI) auf dem Referenzkit M5Stack Core2 für AWS IoT EduKit. Sie lernen, wie Sie das Espressif Voice Assistant SDK (VA-SDK) und Alexa verwenden, um die integrierte LED mithilfe von Alexa Smart Home-Befehlen zu steuern. In diesem Tutorial wird derzeit ein von Espressif bereitgestelltes AWS-Konto verwendet. Dies ist ein *beta*-Port des Espressif VA-SDK für die M5Stack Core2 des AWS IoT EduKit.

Annahmen. Bevor Sie mit diesem Tutorial beginnen, überprüfen Sie die folgenden Voraussetzungen:

1. Sie haben ein [M5Stack Core2 ESP32 IoT-Entwicklungskit für AWS IoT EduKit](https://www.amazon.com/dp/B08VGRZYJR/).
2. Sie haben die Tutorials [**Getting Started**](/de/getting-started.html) und [**Blinky Hello World**](/de/blinky-hello-world.html) abgeschlossen und haben Ihre Entwicklungsumgebung eingerichtet. Sie können Firmware kompilieren und auf das Gerät hochladen.
3. Sie haben mindestens ein grundlegendes technisches Verständnis von AWS IoT-Messaging-Konzepten wie Topics, Veröffentlichungen und Abonnements.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}