+++
pre = "› "
linkTitle = "Linux"
title = "Ubuntu Linux v18.0+ Setup-Anweisungen"
+++

In diesem Abschnitt erfahren Sie, wie Sie Ihren Linux-Computer (Host-Computer) einrichten, um den Code aus dem GitHub-Repository herunterzuladen, den Code anzuzeigen und zu bearbeiten, ihn für die Hardware zu kompilieren und in den Flash-Speicher der Hardware hochzuladen. Diese Installationsschritte reichen für das Tutorial **Erste Schritte** aus, in dem das AWS-Konto und die Dienste von Espressif für die RainMaker-Plattform verwendet werden.

## Abhänigkeiten installieren
Um den Code herunterzuladen, [zu kompilieren](https://de.wikipedia.org/wiki/Compiler), und Skripte auszuführen und [pip](https://pip.pypa.io/en/stable/) zu verwenden, müssen Sie zuerst einige Abhängigkeitsbibliotheken oder -anwendungen installieren. Um den Code aus dem Remote-Code-Repository auf GitHub herunterzuladen, müssen Sie [Git](https://git-scm.com/) installieren. Um den Code zu kompilieren, verwendet Espressif das [cmake](https://cmake.org/)-Build-System. Für die Analysen der Konfiguration der Anwendung wird der [gcc Compiler](https://gcc.gnu.org/onlinedocs/gcc/) verwendet um so die Firmware zu Objektcode zu kompilieren, den der Core2 des AWS IoT EduKit verstehen und ausführen kann. Alle Abhängigkeiten können einfach mit apt über den folgenden Befehl im Terminal installiert werden:
```bash
sudo apt install build-essential python3-pip curl git cmake
```

## Silicon Labs USB-zu-UART-Bridge-Treiberinstallation
Der Core2 des AWS IoT EduKit kommuniziert mit dem Host-Computer über die Silicon Labs CP210x USB-zu-UART-Bridge. Der integrierte CP2104 ist eine USB-zu-UART-Bridge zur Erleichterung der Host-Kommunikation mit dem ESP32-D0WD-Mikrocontroller. Der Mikrocontroller kommuniziert bidirektional über [UART0](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/uart.html), was über einen virtuellen Kommunikationsport des Host-Computers vom CP210x übersetzt wird, der über USB-C eingerichtet ist.

Die Linux-Kernel-Versionen 3.x.x und 4.x.x enthalten bereits die Treiber in ihren Distributionen, die für den CP210x benötigt werden. Standardmäßig ist die serielle Kommunikation jedoch auf Benutzer beschränkt, die Teil der Nutzergruppe sind. Um Ihren Benutzer zur Nutzergruppe hinzuzufügen, öffnen Sie das Terminal und geben Sie Folgendes ein:
```bash
sudo usermod -a -G dialout $USER
```

Sie müssen Ihren Host-Computer *jetzt* neu starten, damit die Berechtigungen angewendet werden.

## Visual Studio-Code Installation
Visual Studio Code (VS Code) ist eine integrierte Open Source-Entwicklungsumgebung (IDE), mit der Sie unter anderem Code anzeigen, bearbeiten und verwalten können. Laden Sie die neuste [Version](https://code.visualstudio.com/) für Ihr Betriebssystem herunter. Informationen zur Behandlung von Problemen bei der Installation oder Verwendung von Visual Studio Code finden Sie in [dessen Dokumentation](https://code.visualstudio.com/docs/setup/setup-overview).

## PlatformIO installieren
[PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) (PIO) bietet eine professionelle Embedded-Entwicklungsplattform, die die Entwicklung eingebetteter Software vereinfacht. Die Visual Studio Code-Erweiterung stellt die Funktionalität der Platform IO-Befehlszeilenschnittstelle (CLI) in einer grafischen Oberfläche bereit. Sie können die Erweiterung herunterladen und [hier](https://platformio.org/install/ide?install=vscode) mehr über PlatformIO lesen.

Sie müssen VS Code neu starten, nachdem die Installation der PlatformIO-Erweiterung abgeschlossen ist.

## Das Code-Repository klonen
Alle Projekte und Dateien befinden sich in einem [GitHub-Repository](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/about-repositories), wo Sie auch den Revisionsverlauf jeder Datei im Repository (repo) anzeigen können. Um den Code zu klonen, den Sie für die Tutorials benötigen, verwenden Sie die PIO-Schnittstelle:
1) Klicken Sie in der VS-Code-Aktivitätsleiste (Menü ganz links) auf das PlatformIO-Logo.
2) Wählen Sie im Menü **Schnellzugriff** von PlatformIO unter **Miscellaneous** die Option **Clone Git Project** aus.
3) Fügen Sie `https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git` in das Textfeld ein und wählen Sie dann den Speicherort aus, an dem Sie das Projekt speichern möchten.
   {{< img "pio-clone_git_project.en.png" "PlatformIO Clone Git Project" "1 - Menü öffnen, 2 - Git Projekt klonen, 3 - Repository URL einfügen" >}}

## Apps herunterladen und installieren
Die ESP RainMaker Apps sind für iOS- und Android-Geräte verfügbar und bieten Wi-Fi-Netzwerkkonfiguration, Benutzererstellung, Benutzerzuordnung und Gerätesteuerung. Die Apps finden Sie hier:
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker), [Direkte APK](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

Wenn Sie kein kompatibles Android- oder iOS-Gerät haben, können Sie die [RainMaker CLI](https://rainmaker.espressif.com/docs/cli-setup.html) verwenden und die bereitgestellten Anweisungen ersetzen.

## Identifizieren des Kommunikationsanschlusses des Geräts
Wenn Sie dies noch nicht getan haben, ist es an der Zeit, das AWS IoT EduKit auszupacken und mit dem mitgelieferten USB-A-auf-USB-C-Kabel an den USB 2.0-Anschluss Ihres Host-Computers anzuschließen. Zusätzlich ist im Paket ein Inbusschlüssel enthalten, mit dem zusätzliche Module installiert werden können (separat erhältlich). Das Gerät sollte sich nach dem Anschließen automatisch einschalten. Wenn dies nicht der Fall ist, drücken Sie den Netzschalter.
![Wie man den M5Stack Core2 für AWS ein- oder ausgeschaltet](linux/core2foraws_power_on_off.jpg?width=500px&classes=shadow)

Wenn das Gerät bereit ist und die Software, die Sie für dieses Tutorial benötigen, installiert ist, identifizieren wir den Port, an dem Ihr Gerät virtuell gemountet ist. So können Sie zukünftig Lese- und Schreibvorgänge für diesen bestimmten Port ausführen.
1) Wählen Sie im Menü **Quick Access** von PlatformIO unter **PIO Home** die Option **Devices** aus.
2) Klicken Sie auf das Symbol neben dem Anschluss mit der Beschreibung „CP2104 USB to UART Bridge Controller“ (normalerweise `/dev/ttyUSB0`), um den Geräteanschluss zu kopieren.

{{% notice note %}}
Wenn Ihr Core2 für AWS IoT EduKit nicht in der Geräteliste angezeigt wird, überprüfen Sie, ob er eingeschaltet ist und Sie das mitgelieferte USB-A-auf-USB-C-Kabel verwenden. Bei einigen USB-C-Hubs treten Kompatibilitätsprobleme beim Aufbau einer seriellen Schnittstelle auf.
{{% /notice %}}

## Weiter
Wenn alles eingerichtet ist und Ihr Host-Computer bereit und in der Lage ist, mit der Core2 für AWS IoT EduKit zu kommunizieren, fahren wir mit dem nächsten Kapitel fort — [**Ausführen des ESP RainMaker Agent**](/de/getting-started/run-rainmaker.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}