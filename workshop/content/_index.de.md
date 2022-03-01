+++
title = "AWS IoT EduKit"
chapter = true
weight = 1
+++

{{% notice info %}}
Herzlichen Glückwunsch an die Gewinner des Hackathons [Hackster.io Reinventing Healthy Spaces](https://www.hackster.io/contests/Healthy-Spaces-with-AWS)! Wir hatten über [76 Einreichungen](https://www.hackster.io/contests/Healthy-Spaces-with-AWS/submissions#challengeNav) von über 674 Teilnehmern, darunter die besten Beiträge 1) [AWS IoT-Impfstoffträger-Kaltbox](https://www.hackster.io/mtrobregado/aws-iot-vaccine-carrier-cold-box-9c7fba), 2) [Automatisches Belüftungssystem mit CO2-Monitoren](https://www.hackster.io/stevekasuya/automatic-ventilation-system-with-co2-monitors-b301f8), und 3) [Tyhac covid-19-Audiodiagnose-Stack](https://www.hackster.io/mick20/tyhac-covid-19-audio-diagnostic-stack-9d5455).
{{% /notice %}}

![EduKit Logo](AWS_IoT_EduKIt_Logo-320px_193px.png)
AWS IoT EduKit ist eine einfache Möglichkeit, mithilfe eines präskriptiven Lernprogramms IoT-Anwendungen mithilfe von AWS-Services zu erstellen.
AWS IoT EduKit hilft Entwicklern — von Studenten bis hin zu erfahrenen Ingenieuren und Fachleuten — praktische Erfahrung beim Erstellen von End-to-End-IoT-Anwendungen zu sammeln, indem ein Referenz-Hardware-Kit mit einer Reihe leicht zu befolgender Anleitungen und Beispielcode kombiniert wird.

### Vorteile von AWS IoT EduKit
#### Vereinfachte Hardwareauswahl
- Das Referenz-Hardware-Kit von AWS IoT EduKit — hergestellt und verkauft von unserem Fertigungspartner M5Stack — ist vollgepackt mit integrierten Funktionen, die eine Vielzahl von IoT-Anwendungen sofort ermöglichen. Entwickler können ihre Fähigkeiten erweitern, um zusätzliche Anwendungsfälle mit Plug & Play-Erweiterungsoptionen abzudecken.
- Das Referenz-Hardware-Kit bietet eine leistungsstarke und sichere Hardware für den Aufbau einer Reihe von IoT-Anwendungen (Einsteiger bis Fortgeschrittene/Professional) zu einem Einstiegspreis. Es wird von einem Espressif ESP32-Mikrocontroller (MCU) betrieben, der mit einem Microchip ATECC608 Trust&GO-Sicherungselement gekoppelt ist. Es ist Alexa-fähig, ist mit mehreren integrierten Peripheriegeräten ausgestattet und eine Vielzahl Erweiterungsmodule für zusätzliche Konnektivitätsoptionen (z. B. LoRaWAN, NB-IoT, LTE) und Plug & Play-Grove-Connector-Peripheriegeräte (z. B. Sensoren, Aktoren) sind seperat erhältlich, um eine breite Abdeckung vieler Use Cases zu erreichen.

#### Unterstützt eine Vielzahl von Software-Frameworks
- Die Referenz-Hardware von AWS IoT EduKit unterstützt eine Vielzahl von Anwendungsframeworks (z. B. FreeRTOS, Arduino, MicroPython), mit denen Entwickler in der Sprache ihrer Wahl coden und beim Aufbau von Cloud-verbundenen eingebetteten Anwendungen auf einer einzigen Hardwareplattform Fortschritte erzielen können.
- AWS IoT EduKit vereinfacht das Onboarding in AWS IoT durch die RainMaker-Plattform und die PlatformIO-Entwicklungsplattform von Espressif. Mit der Espressif RainMaker-Plattform können Sie das Referenz-Hardware-Kit steuern und die Smart-Home-Anwendung ohne AWS-Konto steuern. Die PlatformIO-Entwicklungsplattform vereinfacht die eingebettete Entwicklung, um Code anzuzeigen, zu bearbeiten und zu flashen.

#### Einfacher Zugriff auf Beispiele
- AWS IoT EduKit bietet aktuelle Inhalte und Beispielcode für die gängigsten IoT-Lösungen.
- Mithilfe von [AWS IoT EduKit-Tutorials](https://edukit.workshop.aws/de/getting-started.html) können Entwickler auf eine Vielzahl kostenloser Inhalte zugreifen, um Fachwissen zum Erstellen und Verwalten von IoT-Anwendungen mithilfe von AWS-Services zu erwerben.

### So funktioniert es
Um loszulegen, kaufen Sie Ihr Referenz-Hardware-Kit bei [Amazon.com](https://www.amazon.com/dp/B08VGRZYJR/) (US), [M5Stack shop](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) (CN/Global), [Digi-Key](https://www.digikey.com/en/products/detail/m5stack-technology-co-ltd/K010-AWS/13562927) (AMER/Global), [Mouser Electronics](https://www.mouser.com/ProductDetail/M5Stack/K010-AWS?qs=%2Fha2pyFaduh2vnlTOLWOXVDYhV94RvwKuua4BUEreQw%3D) (AMER/Global), [Switch Science](https://www.switch-science.com/catalog/6784/) (JP), oder [Distrelec](https://www.distrelec.biz/en/esp32-m5core2-iot-development-kit-for-aws-iot-edukit-m5stack-k010-aws/p/30196462) (EU).

Sobald Sie das Hardware-Kit erhalten haben, starten Sie mit den [Ersten Schritten](https://edukit.workshop.aws/de/getting-started.html) und befolgen Sie den Abschnitt „Voraussetzungen“, um die RainMaker Agent-Firmware auf dem Gerät und die Espressif RainMaker Mobile App auf Ihrem Mobiltelefon zu installieren. Mit der Espressif RainMaker Mobile App können Sie die AWS IoT EduKit-Referenz-Hardware steuern und eine Verbindung zu AWS IoT herstellen. Als Nächstes können Sie aus einer Liste kostenloser Projekte auswählen, die auf der Website von AWS IoT EduKit verfügbar sind. Beginnen Sie mit der Entwicklung einer einfachen Connected-Home-Anwendung und gehen Sie im Laufe der Zeit zur Ausführung eines Modells für maschinelles Lernen mit Amazon SageMaker Autopilot oder zum Erstellen einer sprachunterstützten Smart Home-Anwendung mit Alexa Voice Service Integration für AWS IoT über. Um mehr über die Hardwarespezifikationen zu erfahren, besuchen Sie bitte die [M5Stack Docs](https://docs.m5stack.com/#/en/core/core2_for_aws) oder dem [Partner Geräte Katalog](https://devices.amazonaws.com/detail/a3G0h000007djMLEAY).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}