+++
linkTitle = "Windows"
title = "Windows 10 Setup-Anweisungen"
pre = "› "
+++

In diesem Abschnitt erfahren Sie, wie Sie Ihren Windows-Computer (Host-Computer) einrichten, um den Code aus dem GitHub-Repository herunterzuladen. Zudem bearbeiten Sie den Code um ihn für die Hardware zu kompilieren und in den Flash-Speicher der Hardware hochzuladen. Diese Installationsschritte reichen für das Tutorial **Erste Schritte** aus, in dem das AWS-Konto und die Dienste von Espressif für die RainMaker-Plattform verwendet werden.

## Git- und Git-Abhängigkeiten installieren
Um den Code aus dem Remote-Code-Repository auf GitHub herunterzuladen, müssen Sie [Git](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F) installieren, ein weit verbreitetes Versionskontrollsystem. Es wird hauptsächlich für die Verwaltung und Zusammenarbeit von Quellcode verwendet, häufig zum Verfolgen von Dateiänderungen oder zum Verteilen von Code zwischen einem lokalen Computer und einem Remote-Server. Um Git und seine Abhängigkeiten zu installieren, benötigen wir [OpenSSL](https://www.openssl.org/):
1) Laden Sie den [Installer](https://git-scm.com/download/win) für Git herunter und führen Sie ihn aus.
2) Wir empfehlen, dass Sie die Standardoptionen verwenden, aber stellen Sie sicher, dass die OpenSSL-Installation ausgewählt ist.


## Silicon Labs USB-zu-UART-Bridge-Treiberinstallation
Der Core2 des AWS IoT EduKit kommuniziert mit dem Host-Computer über die Silicon Labs CP210x USB-zu-UART-Bridge. Der integrierte CP2104 ist eine USB-zu-UART-Bridge zur Erleichterung der Host-Kommunikation mit dem ESP32-D0WD-Mikrocontroller. Der Mikrocontroller kommuniziert bidirektional über [UART0](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/uart.html), was über einen virtuellen Kommunikationsport des Host-Computers vom CP210x übersetzt wird, der über USB-C eingerichtet ist. Um die virtuelle serielle Schnittstelle mounten und über sie kommunizieren zu können, müssen Sie den entsprechenden Treiber herunterladen und installieren.

1) Stellen Sie sicher, dass das Gerät nicht mit dem Hostcomputer verbunden ist.
2) Laden Sie den macOS Windows Labs CP210x-Treiber [hier](https://www.silabs.com/documents/public/software/CP210x_VCP_Windows.zip) herunter.
3) Extrahieren Sie den Inhalt des Downloads.
   {{% notice note %}}
   Sie müssen den Inhalt des Ordners extrahieren. Der Treiber wird nicht installiert, wenn die ausführbare Datei aus dem Archiv heraus ausführen.
   {{% /notice %}}
4) Führen Sie das **CP210xVCPInstaller_x64.exe**-Installationsprogramm aus.
5) Starten Sie jetzt Ihren Host-Computer neu, um sicherzustellen, dass der Treiber angewendet wird.

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
Die ESP RainMaker Phone Apps sind für iOS- und Android-Telefone verfügbar und bieten Wi-Fi-Netzwerkkonfiguration, Benutzererstellung, Benutzerzuordnung und Gerätesteuerung. Die Apps finden Sie hier:
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker), [Direkte APK](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

Wenn Sie kein kompatibles Android- oder iOS-Gerät besitzen, können Sie die [RainMaker CLI](https://rainmaker.espressif.com/docs/cli-setup.html) verwenden und die bereitgestellten Anweisungen ersetzen.

## Identifizieren des Kommunikationsanschlusses des Geräts
Wenn Sie dies noch nicht getan haben, ist es an der Zeit, den Core2 für das AWS IoT EduKit auszupacken und mithilfe des mitgelieferten USB-A-auf-USB-C-Kabels an den USB 2.0-Anschluss Ihres Host-Computers anzuschließen. Zusätzlich ist im Paket ein Inbusschlüssel enthalten, mit dem Sie zusätzliche Module (separat erhältlich) installieren können. Das Gerät sollte sich nach dem Anschließen automatisch einschalten. Wenn dies nicht der Fall ist, drücken Sie den Netzschalter.
![Wie kann M5Stack Core2 für AWS ein- oder ausgeschaltet werden](macos/core2foraws_power_on_off.jpg? width=500px&classes=shadow)

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