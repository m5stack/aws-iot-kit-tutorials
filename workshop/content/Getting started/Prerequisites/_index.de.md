+++
title = "Voraussetzungen"
weight = 11
pre = "<b>a. </b>"
+++

In diesem Kapitel laden wir die Software herunter und installieren sie, die Sie zum Anzeigen, Bearbeiten, Kompilieren einer binären Firmware und zum Übertragen der Firmware auf das Gerät benötigen. Außerdem werden Sie eine App auf Ihr Telefon herunterladen, um das Gerät zu steuern.

## Silicon Labs USB-zu-UART-Bridge-Treiberinstallation
Laden Sie die SiLabs CP210x-Treiber herunter und installieren Sie sie, damit Ihr Computer mit dem Core2 for AWS IoT EduKit-Gerät kommunizieren kann. Der integrierte CP2104 ist eine USB-zu-UART-Brücke, die die Host-Kommunikation mit dem ESP32-D0WD-Mikrocontroller erleichtert:
{{%expand "macOS 10.9+" %}}
Seit OS X Mavericks hat Apple die notwendigen USB-zu-Seriell-Treiber mitgeliefert und es sind keine weiteren Schritte notwendig. Um zu überprüfen, ob die Treiber installiert und geladen sind und das Gerät zur Programmierung gelesen wird, schließen Sie das Gerät über das mitgelieferte USB-C-Kabel an und führen Sie aus:
```bash
ls -l /dev/cu.S*
```
Wenn Ihr Host-Rechner das Gerät erkennt und mit ihm kommunizieren kann, sehen Sie einen Rückgabewert von `/dev/cu.SLAB_USBtoUART` ausgegeben. Wenn das Ergebnis des obigen Befehls leer zurückgegeben wird, überprüfen Sie zuerst die physische Verbindung. Wenn dies das Problem nicht behebt, folgen Sie den Anweisungen für macOS <= 10.8.
{{% notice info %}}
Unter macOS 10.13 und später kann die Installation der SiLabs-Systemerweiterung blockiert sein. Um die Blockierung aufzuheben, öffnen Sie den Bereich **Systemeinstellungen** <i class="fas fa-arrow-right"></i> **Sicherheit & Datenschutz** Ihres Macs, heben Sie die Blockierung auf, indem Sie auf das <i class="fas fa-lock"></i> und **"Entwickler zulassen"** klicken. Weitere Informationen finden Sie im [Apple Technical Note TN2459](https://developer.apple.com/library/archive/technotes/tn2459/_index.html).
{{% /notice %}}
{{% /expand%}}
{{%expand "macOS <= 10.8" %}}
1) Trennen Sie das USB-C-Kabel, das Ihr Core2 for AWS IoT EduKit-Gerät mit Ihrem Computer verbindet, falls es bereits angeschlossen ist.
2) Laden Sie den [CP210x VCP macOS Treiber](https://www.silabs.com/documents/public/software/Mac_OSX_VCP_Driver.zip) herunter und extrahieren Sie ihn.
3) Erweitern Sie die **SiLabsUSBDriverDisk.dmg** Datei innerhalb des extrahierten Ordners.
4) Öffnen Sie die Anwendung **Install CP210x VCP Driver** und führen Sie das Installationsprogramm aus.
5) Starten Sie Ihren Computer neu und schließen Sie Ihr Core2 for AWS IoT EduKit-Gerät über das mitgelieferte USB-C-Kabel wieder an Ihren Computer an.
{{% /expand%}}
{{%expand "Linux" %}} 
In den Linux-Kernel-Versionen 3.x.x und 4.x.x sind die Treiber bereits Teil der Distribution. Um zu überprüfen, ob sie installiert und geladen sind, führen Sie den Befehl in Ihrem Terminal aus:
```bash
modinfo usbserial
modinfo cp210x
```
Wenn sie *nicht* geladen sind, führen Sie den Befehl aus:
```bash
sudo modprobe usbserial
sudo modprobe cp210x
```
{{% /expand%}}
{{%expand "Windows" %}}
1) Trennen Sie das USB-C-Kabel, das Ihr Core2 for AWS IoT EduKit-Gerät mit Ihrem Computer verbindet, falls es bereits angeschlossen ist.
2) Laden Sie den [CP210x Universal Windows driver](https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip) herunter und extrahieren Sie ihn.
3) Öffnen Sie **CP210xVCPInstaller_x64.exe** und führen Sie das Installationsprogramm aus, um die Treiber zu installieren.
4) Starten Sie Ihren Computer neu und schließen Sie Ihr Core2 for AWS IoT EduKit-Gerät über das mitgelieferte USB-C-Kabel wieder an Ihren Computer an.
   {{% notice warning %}}
   Die Anleitungen wurden nur auf Windows 10 64-Bit getestet. Andere Konfigurationen des Windows-Betriebssystems werden von uns nicht unterstützt.
{{% /notice %}}
{{% /expand%}}

## Visual Studio Code-Installation
Visual Studio Code ist eine quelloffene integrierte Entwicklungsumgebung (IDE), mit der Sie Code anzeigen, bearbeiten, verwalten und mehr können. Laden Sie das neueste [Visual Studio Code](https://code.visualstudio.com/) für Ihr Betriebssystem herunter. Zur Behebung von Problemen bei der Installation oder Verwendung von Visual Studio Code lesen Sie bitte die [zugehörige Dokumentation](https://code.visualstudio.com/docs/setup/setup-overview).

## Installing PlatformIO
[PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) bietet eine professionelle Embedded-Entwicklungsplattform, die die Entwicklung von Embedded-Software vereinfacht. Die Visual Studio Code-Erweiterung stellt die Funktionalität der Platform IO-Befehlszeilenschnittstelle (CLI) in einer grafischen Oberfläche zur Verfügung. Sie können die Erweiterung herunterladen und mehr über PlatformIO [hier](https://platformio.org/install/ide?install=vscode) lesen.

{{%expand "Windows" %}}
{{% notice info %}}
Wenn Sie bereits Espressif IoT Development Framework (ESP-IDF) auf Ihrem Computer installiert haben, wird möglicherweise eine Fehlermeldung über fehlende Dateien angezeigt oder Abhängigkeiten wie **esp-windows-curses** können nicht installiert werden. Dies ist auf Konflikte zurückzuführen. Um das Problem zu diesem Zeitpunkt zu beheben, beziehen Sie sich bitte auf [diesen Beitrag](https://community.platformio.org/t/cant-create-esp-idf-project-correctly-in-platformio/16370/17) im PlatformIO Community Forum.
{{% /notice %}}
{{% /expand%}}

## Herunterladen und Installieren der Telefon-Apps.
Die ESP RainMaker Phone Apps sind für iOS- und Android-Telefone verfügbar und ermöglichen die Konfiguration des Wi-Fi-Netzwerks, die Erstellung von Benutzern, die Zuordnung von Benutzern zu Geräten und die Gerätesteuerung. Die Apps sind hier zu finden:
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker), [Direct APK](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

Wenn Sie kein kompatibles Android- oder iOS-Gerät besitzen, können Sie die [Rainmaker CLI](https://rainmaker.espressif.com/docs/cli-setup.html) und eine Ersatzanleitung verwenden.

## Herunterladen des Codes
Sie können den Code, den Sie in diesem Programm verwenden werden, entweder herunterladen, indem Sie das Repository auf [GitHub](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit) klonen, oder die [zip-Datei](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/archive/master.zip) herunterladen und entpacken. Zum Klonen:
```bash
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```
{{%expand "Windows" %}}
{{% notice info %}}
Unter Windows gibt es Beschränkungen der Dateipfadlänge, die zu Fehlern führen können, je nachdem, wo das Projekt abgelegt ist. Es wird empfohlen, dieses Repo zu klonen oder das Zip mit einer kürzeren anfänglichen Pfadlänge zu entpacken, wenn dieser Fehler auftreten sollte (z. B. **C:\\**).
{{% /notice %}}
{{% /expand%}}

## Anschließen und Einschalten des Geräts
Als Letztes stellen wir sicher, dass das Gerät an den Computer angeschlossen und eingeschaltet ist. Das Gerät sollte sich automatisch einschalten, sobald es eingesteckt und mit einer Stromquelle verbunden ist, aber wenn Sie es einschalten müssen, drücken Sie den Netzschalter.
[So schalten Sie M5Stack Core2 für AWS ein oder aus](prerequisites/core2foraws_power_on_off.jpg?width=500px&classes=shadow).
Wenn das Gerät bereit ist, gehen wir zum nächsten Kapitel - [**Ausführen des ESP RainMaker-Agenten**](/de/getting-started/run-rainmaker.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}