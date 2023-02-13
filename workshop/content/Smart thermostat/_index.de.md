+++
title = "Intelligentes Thermostat"
chapter = true
weight = 30
pre = "<b>3. </b>"
+++

In dieser Lerneinheit konfigurieren Sie Ihre Referenzhardware in einem intelligenten Thermostat, das eines Heizungs-, Lüftungs- und Klimatechnik-System (HVAC) steuert. Ihr intelligentes Thermostat meldet die gemessene Raumtemperatur und den Geräuschpegel an den Geräteschatten ("shadow") seines Geräts in der Cloud. Sie konfigurieren eine serverlose Anwendung, die auf die gemeldeten Messungen wartet, den Status des Thermostats ermittelt, und Befehle an das Gerät zurücksendet.

Annahmen. Bevor Sie mit diesem Tutorial beginnen, überprüfen Sie die folgenden Voraussetzungen:
1. Sie haben ein [M5Stack Core2 ESP32 IoT-Entwicklungskit für das AWS IoT Kit](https://www.amazon.com/dp/B08VGRZYJR/).
2. Sie haben ein AWS-Konto, das keine produktiven Workloads ausführt (d. h. ein Konto, das für Sandbox- und Evaluierungszwecke bereitgestellt ist).
3. Sie haben einen Nutzer oder eine Rolle für das AWS-Konto mit Administratorzugriff.
4. Ihr Core2 des AWS IoT Kit wurde in AWS IoT Core bereitgestellt und kommuniziert bereits über MQTT mit AWS. Beginnen Sie mit dem [**Blinky Hello World**](/de/blinky-hello-world.html) -Tutorial, wenn dies noch nicht passiert ist.
5. Sie haben mindestens ein grundlegendes technisches Verständnis von AWS IoT-Messaging-Konzepten wie Topics, Veröffentlichungen und Abonnements.

Ziele. Am Ende dieses Labors sollten Sie folgendes wissen:
1. So erfassen Sie Temperatur und Schallpegel vom Core2 des AWS IoT Kit-Gerät.
2. So veröffentlichen Sie Temperatur- und Schallmessungen vom Gerät zum AWS IoT Core.
3. So melden Sie Messwerte an den Geräteschatten.
4. So führen Sie Nachrichten-Transformationen mit AWS IoT-Core Regeln durch.
5. So erstellen Sie eine serverlose Anwendung, die auf Eingaben reagiert und komplexe Events erkennt.
6. So senden Sie Befehle durch den Geräteschatten an Ihr Gerät.

Um mit diesem Tutorial zu beginnen, lesen Sie das erste Kapitel [**Einführung**](/de/smart-thermostat/introduction.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}