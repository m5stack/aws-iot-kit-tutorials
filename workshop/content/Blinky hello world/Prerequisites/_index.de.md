+++
title = "Voraussetzungen"
weight = 10
pre = "<b>a. </b>"
+++

In diesem Kapitel installieren Sie die erforderliche Software für dieses und alle folgenden Tutorials. Beginnend mit den SiLabs CP210x-Treibern zur Kommunikation mit dem Core 2 for AWS IoT EduKit über USB, dann die ESP-IDF-Toolchain für die integrierte ESP32-D0WD-Mikrocontrollereinheit (MCU). Sie werden auch Miniconda installieren, um Ihre Python-Versionen und Ihre Abhängigkeiten in isolierten virtuellen Umgebungen zu verwalten, um Konflikte zu vermeiden. Außerdem werden wir die AWS-Befehlszeilenschnittstelle (CLI) herunterladen, installieren und konfigurieren. In diesem Lernprogramm wird davon ausgegangen, dass Sie ein [AWS-Konto](https://console.aws.amazon.com/) haben.

## Silicon Labs USB-zu-UART-Bridge-Treiberinstallation
Laden Sie die SiLabs CP210x-Treiber herunter und installieren Sie sie, damit Ihr Computer mit dem Core2 for AWS IoT EduKit-Gerät kommunizieren kann. Der integrierte CP2104 ist eine USB-zu-UART-Brücke, die die Host-Kommunikation mit dem ESP32-D0WD-Mikrocontroller erleichtert:
{{%expand "macOS 10.9+" %}}
Seit OS X Mavericks hat Apple die notwendigen USB-zu-Seriell-Treiber mitgeliefert und es sind keine weiteren Schritte notwendig. Um zu überprüfen, ob die Treiber installiert und geladen sind und das Gerät zur Programmierung gelesen wird, schließen Sie das Gerät über das mitgelieferte USB-C-Kabel an und führen Sie aus:
```bash
ls -l /dev/cu.S*
```
Wenn Ihr Host-Rechner das Gerät erkennt und mit ihm kommunizieren kann, sehen Sie einen Rückgabewert von `/dev/cu.SLAB_USBtoUART` ausgegeben. Wenn das Ergebnis des obigen Befehls leer zurückgegeben wird, überprüfen Sie zuerst die physische Verbindung. Wenn dies das Problem nicht behebt, folgen Sie den Anweisungen für macOS <= 10.8.
{{% notice info %}}
Unter macOS 10.13 und später kann die Installation der SiLabs-Systemerweiterung blockiert sein. Um die Blockierung aufzuheben, öffnen Sie den Bereich **Systemeinstellungen** <i class="fas fa-arrow-right"></i> **Sicherheit & Datenschutz** Ihres Macs, heben Sie die Blockierung auf, indem Sie auf das <i class="fas fa-lock"></i> klicken, **den Entwickler zulassen** und dann die Blockierung durch Klicken auf das <i class="fas fa-lock-open"></i> wieder aufheben. Weitere Informationen finden Sie im [Apple Technical Note TN2459](https://developer.apple.com/library/archive/technotes/tn2459/_index.html).
{{% /notice %}}
{{% /expand%}}
{{%expand "macOS <= 10.8" %}}
1) Trennen Sie das USB-C-Kabel, das Ihr Core2 for AWS IoT EduKit-Gerät mit Ihrem Computer verbindet, falls es bereits angeschlossen ist.
2) Laden Sie den [CP210x VCP macOS Treiber](https://www.silabs.com/documents/public/software/Mac_OSX_VCP_Driver.zip) herunter und extrahieren Sie ihn.
3) Öffnen Sie die **SiLabsUSBDriverDisk.dmg** Datei innerhalb des extrahierten Ordners.
4) Öffnen Sie die Anwendung **Install CP210x VCP Driver** und führen Sie das Installationsprogramm durch.
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
{{%expand "Windows (64-bit)" %}}
1) Trennen Sie das USB-C-Kabel, das Ihr Core2 for AWS IoT EduKit-Gerät mit Ihrem Computer verbindet, falls es bereits angeschlossen ist.
2) Laden Sie den [CP210x Universal Windows driver](https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip) herunter und extrahieren Sie ihn.
3) Öffnen Sie **CP210xVCPInstaller_x64.exe** und führen Sie das Installationsprogramm aus, um die Treiber zu installieren.
4) Starten Sie Ihren Computer neu und schließen Sie Ihr Core2 for AWS IoT EduKit-Gerät über das mitgelieferte USB-C-Kabel wieder an Ihren Computer an.
{{% notice warning %}}
Die Anleitungen wurden nur auf der 64-Bit-Version von Windows 10 getestet. Wir unterstützen keine andere Konfiguration des Windows-Betriebssystems.
{{% /notice %}}
{{% /expand%}}

## Installation von ESP-IDF v4.2

Die von Ihnen verwendete **M5Stack Core2 for AWS** Hardware ist mit einer Espressif-ESP32-D0WD-Mikrocontrollereinheit (MCU) ausgestattet. Mit dem ESP-IDF (Espressif IoT Development Framework) können Sie Firmware auf einem ESP32-Board konfigurieren, erstellen und flashen. Im &quot;Getting Started&quot;-Tutorial haben Sie PlatformIO verwendet, um die Kompilierung und das Flashen der Firmware auf dem Gerät zu vereinfachen - es bietet eine GUI für die Aufrufe von ESP-IDF Version 4.1 und anderen Tools im Hintergrund. Für die folgenden Schritte verwenden Sie ESP-IDF *Version 4.2* aufgrund von Erweiterungen der mbedTLS-Bibliothek und einer verbesserten Integration des Secure Elements.

{{%expand "macOS" %}}
Kopieren Sie den Befehlsblock unten und fügen Sie ihn ein, um:
1) Installieren Sie Homebrew, wenn Sie es noch nicht haben.
2) Installieren Sie cmake, ninja und das DFU-Dienstprogramm mit Homebrew.
3) Klonen Sie das ESP-IDF in das Verzeichnis `$HOME/esp/` (der vollständige ESP-IDF-Pfad wird **$HOME/esp/esp-idf** sein).
4) Installieren Sie die ESP-IDF-Abhängigkeiten.
5) Überprüfen Sie Ihre Installation und exportieren Sie die ESP-IDF Umgebungsvariablen in Ihren **$PATH**.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install cmake ninja dfu-util
mkdir $HOME/esp
cd $HOME/esp
git clone -b release/v4.2 --recursive https://github.com/espressif/esp-idf.git
cd $HOME/esp/esp-idf
. $HOME/esp/esp-idf/install.sh
. $HOME/esp/esp-idf/export.sh
```
{{% /expand%}}
{{%expand "Linux" %}}
1) Installieren Sie alle Ihre Abhängigkeiten.
   - CentOS 7:
   ```bash
   sudo yum install git wget flex bison gperf cmake ninja-build ccache dfu-util
   ```
   - Ubuntu und Debian:
   ```bash
    sudo apt-get install git wget flex bison gperf python-setuptools cmake ninja-build ccache libffi-dev libssl-dev dfu-util
    ```
   - Arch:
    ```bash
    sudo pacman -S --needed gcc git make flex bison gperf cmake ninja ccache dfu-util
    ```
2) Klonen Sie das ESP-IDF in das Verzeichnis `$HOME/esp/` (der vollständige ESP-IDF Pfad ist $HOME/esp/esp-idf).
3) Installieren Sie die ESP-IDF-Abhängigkeiten.
4) Überprüfen Sie Ihre Installation und exportieren Sie die ESP-IDF Umgebungsvariablen in Ihren **$PATH**.

```bash
mkdir $HOME/esp

cd $HOME/esp

git clone -b release/v4.2 --recursive https://github.com/espressif/esp-idf.git

cd $HOME/esp/esp-idf

. $HOME/esp/esp-idf/install.sh

. $HOME/esp/esp-idf/export.sh
```
{{% /expand%}}
{{%expand "Windows (64-bit)" %}}
1) Laden Sie den [ESP-IDF Tools Installer](https://dl.espressif.com/dl/esp-idf-tools-setup-2.3.exe) herunter und führen Sie es aus, um **ESP-IDF v4.2** und alle anderen Abhängigkeiten zu installieren.
2) Wählen Sie in der Phase **Python-Auswahl** des Installationsprogramms die Option **Python 3.7 installieren**. Wenn Python 3.7 bereits installiert ist, steht Ihnen die Installationsoption möglicherweise nicht zur Verfügung - wählen Sie Ihr installiertes Python 3.7 (verwenden Sie *nicht* das mit Anaconda/miniconda installierte Python, falls diese Option verfügbar ist).
3) Wählen Sie im Schritt **Download ESP-IDF** des herunterzuladenden Installationsprogramms **release/v4.2 (release branch)**.
4) Starten Sie Ihren Rechner neu, falls Sie dies noch nicht getan haben, nachdem das Installationsprogramm ausgeführt wurde.
5) `cd %userprofile%\Desktop\esp-idf`
6) Führen Sie das ESP-IDF-Installationsskript aus, um sicherzustellen, dass alles richtig eingestellt ist: `install.bat`
7) Führen Sie das Export-Skript aus und fügen Sie die ESP-IDF-Tools zu Ihrem Pfad hinzu: `export.bat`
   {{% /expand%}}

## Installieren und Konfigurieren der AWS CLI Version 2

### AWS CLI-Installation

Die AWS-Befehlszeilenschnittstelle (CLI) ist ein einheitliches Tool zur Verwaltung Ihrer AWS-Services. Mit nur einem Tool, das Sie herunterladen und konfigurieren müssen, können Sie mehrere AWS-Services über die Befehlszeile steuern und durch Skripte automatisieren. Um die AWS CLI konfigurieren zu können, benötigen Sie zunächst ein AWS-Konto. Bitte [melden Sie sich an](https://signin.aws.amazon.com/signin) oder [erstellen Sie](https://portal.aws.amazon.com/billing/signup#/start) zunächst ein Konto, bevor Sie fortfahren. Nachdem Sie sich angemeldet haben, folgen Sie den offiziellen [AWS CLI-Installationsanweisungen für Ihr Betriebssystem](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

### AWS CLI-Konfiguration

Sobald Sie das installiert haben, ist es an der Zeit, es für Ihr Konto und Ihre Region einzustellen. Es ist wichtig zu beachten, dass die Region, die Sie derzeit verwenden, konsistent bleibt - für die Zwecke dieses Tutorials standardisieren wir **us-west-2**. Das Ändern von Regionen kann zu anderen Herausforderungen in den nachfolgenden Schritten führen. Eine Liste der verfügbaren Services finden Sie in der [AWS Regional Services List,](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) um die regionale Serviceverfügbarkeit anzuzeigen.

Verwenden Sie den Befehl:
```bash
aws configure
```
Schritte zum Abrufen der Anmeldeinformationen für die AWS CLI finden Sie in der Schnellkonfigurationsdokumentation. Um die AWS-Zugangsschlüssel-ID und den geheimen AWS-Zugangsschlüssel zu erhalten, müssen Sie [Ihrem IAM-Benutzer die](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) erforderlichen [Berechtigungen](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) erteilen. Ihre Eingaben sollten wie die folgenden aussehen:
```
AWS Access Key ID [None]: EXAMPLEKEYIDEXAMPLE
AWS Secret Access Key [None]: EXAMPLEtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## Einrichtung und Installation von Miniconda
Miniconda ist eine Bootstrap-Version des vollwertigen Anaconda, die die Verwaltung von Paketen und virtuellen Python-Umgebungen vereinfacht. Viele Installationsfehler sind auf inkompatible Python-Versionen (z. B. Python 2.x oder 3.8.x), auf die falsche Python-Version in der PATH-Variable oder auf Abhängigkeits-Konflikte zurückzuführen. Es wird *dringend empfohlen*, immer Conda zu verwenden, um Ihre Python-Entwicklungsumgebung zu verwalten.

Gehen Sie auf [https://docs.conda.io/en/latest/miniconda.html](https://docs.conda.io/en/latest/miniconda.html) und laden Sie das entsprechende Installationsprogramm für Ihr Betriebssystem herunter. Folgen Sie den offiziellen Miniconda-Installationsanweisungen und kehren Sie dann zu dieser Seite zurück.

{{%expand "macOS and Linux" %}}
1) Nachdem miniconda installiert ist, beenden Sie Ihre Terminalanwendung und öffnen Sie sie erneut, damit die Quelldateien hinzugefügt werden und Ihr PATH die Umgebungsvariablen enthält, die Sie benötigen werden.
2) Erstellen Sie eine neue conda-Umgebung, um Ihre Arbeit zu isolieren und Abhängigkeits-Konflikte zu vermeiden. Nachdem Sie den folgenden Befehl ausgeführt haben, geben Sie **y** ein, um die notwendigen Abhängigkeiten für die Umgebung zu installieren.
   ```bash
   conda create -n edukit python=3.7
   ```
3) Nachdem Sie die neue Umgebung erfolgreich erstellt haben, wechseln Sie aus der *base* conda-Umgebung in die neu erstellte Umgebung. Sie wissen, dass Sie aktiviert sind, weil Sie **(edukit)** am Anfang der aktuellen Eingabeaufforderung sehen.
   ```bash
   conda activate edukit
   ```
4) Führen Sie das Skript ESP-IDF toolchain export aus, um die ESP-Tools zu Ihrem Pfad hinzuzufügen.
    ```bash
    . $HOME/esp/esp-idf/export.sh
    ```
{{% /expand%}}
{{%expand "Windows (64-bit)" %}}
1) Nachdem das miniconda-Installationsprogramm beendet ist, starten Sie Ihren Rechner neu.
2) Öffnen Sie die Anaconda-Eingabeaufforderung, indem Sie auf **Startmenü** → **Anaconda-Eingabeaufforderung** gehen.
3) Erstellen Sie eine neue Anaconda-Umgebung, um Ihre Arbeit zu isolieren und Abhängigkeits-Konflikte zu vermeiden. Nachdem Sie den folgenden Befehl ausgeführt haben, geben Sie **y** ein, um die notwendigen Abhängigkeiten für die Umgebung zu installieren.
   ```bash
   conda create -n edukit python=3.7
   ```
4) Nachdem Sie die neue Umgebung erfolgreich erstellt haben, wechseln Sie aus der *Basis* conda-Umgebung in die neu erstellte Umgebung. Sie wissen, dass Sie aktiviert sind, weil Sie **(edukit)** am Anfang der aktuellen Eingabeaufforderung sehen.
   ```bash
   conda activate edukit
   ```
5) Führen Sie das Export-Skript aus und fügen Sie die ESP-IDF-Tools zu Ihrem Pfad hinzu: 
   ```bash
   %userprofile%\Desktop\esp-idf\export.bat
   ```
{{% /expand%}}

{{% notice note %}}
Wenn Sie Ihre Shell schließen oder eine neue Shell öffnen, müssen Sie erneut `conda activate edukit` eingeben, um die virtuelle Umgebung zu reaktivieren und die `export.sh` (macOS/Linux) bzw. `export.bat` (Windows) von ESP-IDF ausführen, um die ESP-IDF-Tools jedes Mal neu zu Ihrem Pfad hinzuzufügen.
{{% /notice %}}

## Klonen des Code-Repositorys
Wenn Sie das Code-Repository aus der Anleitung [&quot;Erste Schritte - Voraussetzungen&quot;](/de/getting-started/prerequisites.html) verlegt haben, können Sie das auf [GitHub](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit) gehostete Repository klonen oder die [Zip-Datei](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/archive/master.zip) herunterladen und entpacken. So klonen Sie:
```bash
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```
{{%expand "Windows" %}}
{{% notice info %}}
Windows hat Beschränkungen der Dateipfadlänge, die je nach Speicherort des Projekts, zu Fehlern führen können. Es wird empfohlen, dieses Repo zu klonen oder die Zip-Datei mit einer kürzeren anfänglichen Pfadlänge zu entpacken, wenn dieser Fehler auftreten sollte (z. B. **C:\\**).
{{% /notice %}}
{{% /expand%}}

Nachdem alles installiert und konfiguriert ist, gehen wir zum nächsten Kapitel über: [Gerätebereitstellung](/de/blinky-hello-world/device-provisioning.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}