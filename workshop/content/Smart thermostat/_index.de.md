+++
title = "Intelligentes Thermostat"
chapter = true
weight = 30
pre = "<b>3. </b>"
+++

In diesem Lernprogramm konfigurieren Sie Ihre Referenzhardware zu einem intelligenten Thermostat, der ein fiktives HLK-System (Abk. für Heizung, Lüftung, Klima, engl. HVAC) steuert. Ihr intelligenter Thermostat meldet die gemessene Raumtemperatur und den Geräuschpegel an seinen Geräteschatten in der Cloud. Sie werden auch eine serverlose Anwendung konfigurieren, die auf die gemeldeten Messungen hört, den Zustand bestimmt, auf den der Thermostat eingestellt werden soll, und Befehle an das Gerät zurücksendet, die ihm sagen, was es tun soll.

Vorraussetzungen. Bevor Sie mit diesem Lernprogramm beginnen, überprüfen Sie die folgenden Voraussetzungen:

1. Sie haben ein [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT EduKit](https://www.amazon.com/dp/B08NP5LVFH).
2. Sie haben ein AWS-Konto, auf dem keine Produktions-Workloads ausgeführt werden (d. h. ein Konto, das für Sandbox- und Evaluierungszwecke sicher ist).
3. Sie haben eine Benutzeranmeldung oder eine Rolle für das AWS-Konto mit Administratorzugriff.
4. Ihr "Core2 for AWS IoT EduKit" wurde in AWS IoT Core bereitgestellt und kommuniziert bereits mit AWS über MQTT. Beginnen Sie mit dem [**Blinky Hello World**](/de/blinky-hello-world.html), wenn Sie die Bereitstellung und den Aufbau der Konnektivität noch nicht abgeschlossen haben.
5. Sie haben ein grundlegendes technisches Verständnis von AWS IoT-Messaging-Konzepten wie Topics, Publishing und Subscribing.

Lernziele. Am Ende dieses Labors sollten Sie wissen:

1. Wie man Temperatur- und Schallpegel mit dem "Core2 for AWS IoT EduKit"-Gerät erfasst.
2. Wie man Temperatur- und Schallmessungen vom Gerät in AWS IoT Core veröffentlicht.
3. Wie man Messwerte an den Geräteschatten meldet.
4. Wie man Nachrichtentransformationen mit der AWS IoT Core Rules Engine durchführt.
5. Wie man eine serverlose Anwendung erstellt, die auf Eingaben reagiert und komplexe Ereignisse erkennt.
6. Wie man Befehle über den Schatten an ein Gerät sendet.

Um mit diesem Lernprogramm zu beginnen, gehen Sie zum ersten Kapitel, [**Einführung**](/de/smart-thermostat/introduction.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}