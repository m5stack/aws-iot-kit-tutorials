+++
title = "Voraussetzungen"
weight = 10
pre = "<b>a. </b>"
+++

In diesem Kapitel laden Sie die [AWS CLI](https://aws.amazon.com/cli/) für das Betriebssystem Ihrer Host-Maschine herunter und installieren sie, erhalten AWS [Identity and Access Management](https://aws.amazon.com/iam/) (IAM) Zugriffsrechte, um Dienste mit der AWS CLI zu verwalten. Sie konfiguriren die AWS CLI und testen diese Funktonalität. In diesem Tutorial wird vorausgesetzt, dass Sie ein [AWS-Konto](https://console.aws.amazon.com/console/home) haben und [Ihre Umgebung eingerichtet haben](../getting-started/prerequisites.html). Wenn Sie die AWS CLI (Version 1 oder Version 2) bereits auf Ihrem Computer installiert und konfiguriert haben, fahren Sie mit dem [Testabschnitt](#testen-der-aws-cli) fort.

## Öffnen Sie das PlatformIO CLI Terminal-Fenster
In den [Ersten Schritten](../getting-started.html) haben Sie PIO und das PIO-Terminalfenster installiert und verwendet. Es ist wichtig, das PIO-Terminalfenster weiterhin für alle nachfolgenden Schritte zu verwenden. Das PIO-Terminalfenster hält zusätzliche Anwendungen und Bibliotheken vor, die Ihr Standardterminal/ihre Eingabeaufforderung möglicherweise nicht hat.
Wenn Sie VS-Code geschlossen haben oder den Terminal-Viewport mit der PlatformIO CLI nicht in VS-Code geladen haben, führen Sie nach dem Öffnen von VS-Code die folgenden Schritte aus:
1) Klicken Sie auf das **PlatformIO-Logo** in der VS-Code-Aktivitätsleiste (Menü ganz links).
2) Wählen Sie im Menü **Quick Access** unter **Miscellaneous** die Option **New Terminal**. Das Terminal-Darstellungsfenster sollte mit einem neuen Terminal mit der Bezeichnung **PlatformIO CLI** geladen werden.
   {{< img "pio-new_terminal-alexa_intro.en.png" "PlatformIO CLI terminal in VS Code" "1 - Menü öffnen, 2 - Neues PIO Terminal öffnen, 3 - Kontrollieren, dass es eine 'PlatformIO CLI' Terminal Sitzung ist">}}

## AWS CLI herunterladen und installieren
Das AWS Command Line Interface (CLI) ist ein einheitliches Tool zur Verwaltung Ihrer AWS-Services. Mit nur einem Tool können Sie mehrere AWS-Services über die Kommandozeile steuern und sie mithilfe von Skripten automatisieren. Um die AWS CLI konfigurieren zu können, benötigen Sie zunächst ein AWS-Konto. Bitte [melden Sie sich in der AWS-Konsole an](https://console.aws.amazon.com/console/home) oder [erstellen Sie ein AWS-Konto](https://portal.aws.amazon.com/billing/signup#/start), bevor Sie fortfahren.

{{%expand "Ubuntu Linux v18.0+ (64-bit)" %}}
Laden Sie die neueste AWS CLI Version 2 für 64-Bit-Linux herunter und installieren Sie sie:
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```
{{% /expand%}}

{{%expand "MacOS 10.14+" %}}
1) Laden Sie das [neueste AWS CLI Version 2](https://awscli.amazonaws.com/AWSCLIV2.pkg) für macOS herunter.
2) Doppelklicken Sie auf die heruntergeladene Installationsdatei und befolgen Sie die Anweisungen auf dem Bildschirm, um sie auf dem Host-Computer zu installieren. Wir empfehlen Ihnen, die AWS CLI für alle Benutzer zu installieren. Wenn Sie dies jedoch nicht möchten, befolgen Sie [diese Anweisungen](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-mac.html#cliv2-mac-install-gui), um einen Symlink zu erstellen und die Datei zu Ihrem Pfad hinzuzufügen.
   {{% /expand%}}

{{%expand "Windows 10 (64-bit)" %}}
1) Laden Sie das [neueste AWS CLI Version 2](https://awscli.amazonaws.com/AWSCLIV2.msi) für Windows herunter.
2) Doppelklicken Sie auf die heruntergeladene Installationsdatei und befolgen Sie die Anweisungen auf dem Bildschirm, um sie auf dem Host-Computer zu installieren.
   {{% /expand%}}

## Erhalten Sie IAM User Berechtigungen
[IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) ist ein Webservice, mit dem Sie den Zugriff auf AWS-Ressourcen sicher kontrollieren können. Wir empfehlen Ihnen, zuerst [einen Administrator-Benutzer zu erstellen](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html), anstatt Ihr root-Benutzerkonto zu verwenden.

Um die Zugangsdaten Ihres IAM-Benutzers abzurufen, folgen Sie der [offiziellen Dokumentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config).

## Konfigurieren der AWS CLI
Nachdem die AWS CLI installiert ist und die IAM-Berechtigungen zur Hand sind, ist es an der Zeit, die AWS CLI zu konfigurieren. Eine der Einstellungen, die Sie konfigurieren, ist die [AWS Region](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html). Es ist wichtig zu beachten, dass die Region, die Sie derzeit verwenden, konsistent bleibt. Für die Zwecke dieses Tutorials standardisieren wir auf**us-west-2**. Die Verwendung einer anderen Region oder das versehentliche Ändern der Region kann zu anderen Herausforderungen führen, z. B. [Verfügbarkeit regionaler Dienste](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/).

Um die AWS CLI auf Ihrem Host-Computer zu konfigurieren, geben Sie im Terminal den folgenden Befehl ein:
```bash
aws configure
```

Die CLI fordert Sie auf, vier Parameter einzugeben. Die Felder sollten ähnlich wie unten ausgefüllt werden, mit der entsprechenden "Access Key ID" und dem "secret Access Key", die zuvor für Ihren IAM-Benutzer erstellt wurden:
```bash
AWS Access Key ID [None]: EXAMPLEKEYIDEXAMPLE
AWS Secret Access Key [None]: EXAMPLEtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## Testen der AWS CLI
Nachdem alles wie oben beschrieben konfiguriert ist, ist es jetzt an der Zeit, Ihre AWS CLI zu testen, um sicherzustellen, dass sie ordnungsgemäß funktioniert. Zunächst überprüfen Sie, ob die CLI installiert ist, und überprüfen dann die Konfiguration.

Um zu überprüfen, ob die CLI korrekt installiert ist, verwenden wir die Versionsoption. Bei einer erfolgreichen Installation wird die AWS CLI-Version ausgegeben (wenn Sie Fehler erhalten, lesen Sie den [Leitfaden zur Fehlerbehebung](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)):
```bash
aws --version
```

Als Nächstes überprüfen Sie, ob die AWS CLI mit Ihren IAM-Anmeldeinformationen und der Region US West (Oregon) konfiguriert ist. Der Befehl, den Sie ausführen, überprüft Ihren [MQTT-Broker](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html)-Endpunkt auf AWS IoT. Sie sollte eine Adresse mit dem Muster `xxxxxxxx-ats.iot.us-west-2.amazonaws.com` sehen. Wenn Sie Fehler erhalten, lesen Sie den [Leitfaden zur Fehlerbehebung](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html).
```bash
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```


Nachdem alles installiert und konfiguriert ist, fahren wir mit dem nächsten Kapitel [**Gerätebereitstellung**](device-provisioning.html) fort.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}