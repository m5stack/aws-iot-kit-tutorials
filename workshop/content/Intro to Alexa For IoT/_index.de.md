+++
title = "Intro zu Alexa für IoT"
chapter = true
weight = 50
pre = "<b>5. </b>"
+++

In diesem Tutorial implementieren Sie die Alexa Voice Service Integration für AWS IoT (AFI) auf dem Referenz-Hardware-Kit M5Stack Core2 for AWS IoT EduKit. Sie lernen, wie Sie das Espressif Voice Assistant SDK (VA-SDK) und Alexa verwenden, um die Onboard-LED mit Alexa Smart Home-Befehlen zu steuern. Dieses Tutorial verwendet derzeit ein von Espressif bereitgestelltes AWS-Konto. Dies ist ein *Beta*-Port des Espressif VA-SDK für die M5Stack Core2 for AWS IoT EduKit-Referenzhardware.

Vorraussetzungen. Bevor Sie mit diesem Tutorial beginnen, überprüfen Sie die folgenden Voraussetzungen:

1. Sie haben ein [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT EduKit](https://www.amazon.com/dp/B08NP5LVFH).
2. Sie haben die erforderliche Toolchain und abhängige Software installiert, um Firmware für Ihre "M5Stack Core2 for AWS IoT EduKit"-Referenzhardware zu erstellen und zu flashen, und wissen, an welchen seriellen Port Ihr "Core2 for AWS IoT EduKit"-Gerät angeschlossen ist. Es wird dringend empfohlen, zuerst das vollständige [**Blinky Hello World**](/de/blinky-hello-world.html) zu absolvieren.
3. Sie haben zumindest ein grundlegendes technisches Verständnis von AWS IoT-Messaging-Konzepten wie Topics, Publishing und Subscribing.
4. Sie haben eine IDE wie Visual Studio Code installiert, wie im Tutorial [**Erste Schritte**](/de/getting-started.html) beschrieben.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}