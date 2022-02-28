+++
title = "Erstellen und Testen des AFI"
weight = 30
pre = "<b>c. </b>"
+++

## Einführung in das Kapitel
In diesem Kapitel erstellen wir die portierte ESP-VA-SDK-Firmware, flashen sie auf das Gerät, konfigurieren das WLAN und autorisieren das Gerät für Ihr Alexa-Konto. Anschließend testen wir einige der Smart-Home-Funktionen mithilfe von Alexa-Sprachbefehlen, die in dieser Beta-Version von AFI verfügbar sind.

## Flashen Sie die Firmware
Verwenden Sie die PlatformIO CLI, um Ihre Firmware zu kompilieren, die Firmware hochzuladen und die serielle Ausgabe Ihres Geräts zu überwachen. Das Erstellen und Flashen der App dauert einige Zeit, aber danach sollten Sie den Stream der Geräteprotokolle in Ihrem Terminal sehen. Wenn Sie den Fehler erhalten, dass der Anschluss nicht automatisch erkannt wird, folgen Sie den [Schritten zur Identifikation des seriellen Ports auf dem Host-Computer](../getting-started/prerequisites/windows.html#Identifizieren-des-Kommunikationsanschlusses-des-Geräts). Sie können den seriellen Monitor mit der Tastenkombination **Strg** + **C** schließen:
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```

## Stellen Sie das Gerät bereit
Um das Gerät bereitzustellen, müssen Sie Ihre Anmeldeinformationen für das WLAN-Netzwerk konfigurieren und die Anwendung mit Ihrem Alexa-Konto mithilfe der ESP Alexa Telefonanwendung autorisieren.

Laden Sie die Anwendung aus Ihrem mobilen App Store herunter.
[iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534)/[Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

Schritte zur Bereitstellung:
1. Starten Sie die App.
- Stellen Sie sicher, dass Sie Bluetooth aktiviert haben und dass die App über die entsprechenden Berechtigungen für den Zugriff auf Bluetooth verfügt.
2. Wählen Sie die Option **Neues Gerät hinzufügen**.
3. Ihr Gerät wird in der App angezeigt.
- Sie sehen ein Modal **Proof of Possession** mit einem Wert von `abcdf1234` oder einem ähnlichen Popup. Drücken Sie einfach **Fertig**, um fortzufahren.
4. Nachdem Sie eine Verbindung zum Gerät hergestellt haben, melden Sie sich bei Ihrem Amazon-Konto an.
5. Wählen Sie das WLAN-Netzwerk aus und geben Sie die Anmeldeinformationen ein.
6. Nach einer erfolgreichen WLAN-Verbindung sehen Sie eine Liste mit Beispieläußerungen.

## Alexa verwenden
Nachdem die vorherigen Schritte abgeschlossen sind, werden auf Ihrem seriellen Monitor eine Reihe von Protokollen angezeigt, darunter einige wie die folgenden:
```bash
I (17325) [http_transport]: Subscribing /capabilities/acknowledge...
I (17535) [http_transport]: Subscribing /connection/fromservice...
I (17735) [http_transport]: Subscribing /directive...
I (17945) [http_transport]: Subscribing /speaker...
...
I (20735) [directive_proc]: Name: EndpointForwarding
...
I (22675) [directive_proc]: Name: SetAttentionState
...
E (22685) [app_va_cb]: Enabling Mic
```

Um mit Alexa zu interagieren, müssen Sie dem Gerät *Alexa* sagen. Dadurch wird die **Espressif Wake Word Engine**, die auf dem Gerät ausgeführt wird, in den Aufmerksamkeitszustand **LISTENING** versetzt. Ausführliche Informationen zu den verschiedenen Aufmerksamkeitszuständen finden Sie in unserer [Dokumentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/ux-design-attention.html#states). Weitere Informationen zur Audioaufnahme finden Sie in der [SpeechRecognizer-API-Dokumentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/avs-speechrecognizer-concepts.html)

{{< img "speechrecognizer-state.en.png" "Audio Capture Speech Recognizer Attention States" >}}

{{% notice info %}}
Genau wie bei jedem Alexa-Gerät wartet das Gerät NUR auf das Schlüsselwort „Alexa“, wenn es sich im Ruhezustand befindet. Erst wenn das Keyword ausgelöst wurde, streamt das Gerät Audio in die Cloud.
{{% /notice %}}

Probieren Sie verschiedene Äußerungen an Alexa aus - die seitlichen LEDs sollten blau leuchten, wenn **Alexa** zu hören ist (wenn Alexa nicht „aufwacht“, sprechen Sie näher am Gerät):
* _Alexa, wie viel Uhr ist es?_
* _Alexa, erzähl mir einen Witz?_
* _Alexa, schalte alle Lichter an_ (Funktioniert nur, wenn Sie bereits einige Alexa Smart Home-Geräte auf demselben Konto haben)
  {{< img "alexa-time.en.webp" "Alexa, what time is it?" "Person: Alexa, wie viel Uhr ist es?, Alexa: Es ist 1:52p.m.">}}

## Alexa Smart Home-Funktionen testen (Beta)
Das AFI-Gerät verfügt über **Alexa Built-In**, was bedeutet, dass Sie direkt auf dem Gerät mit Alexa sprechen können und Alexa mit Sprache auf dem Gerät reagiert. Diese Version von AFI von Espressif unterstützt auch Alexa Smart Home-Befehle als Beta-Funktion, mit der Sie Attribute auf dem Gerät steuern können.

Die Alexa für AWS IoT-Beispielanwendung erstellt in Ihrer Alexa-App ein virtuelles Gerät mit dem Namen **Light**, das zwei Schnittstellen unterstützt:

* [PowerController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-powercontroller.html) zum Ein- und Ausschalten des Lichts.
* [RangeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-rangecontroller.html) zum Einstellen der Helligkeit des Geräts.

![The device named "Light" should show up in your Alexa App](building-and-testing-afi/alexa_app-light_device.en.png?height=500px&classes=shadow)

Da es sich um ein virtuelles Gerät handelt, wird der aktualisierte Status auf dem Bildschirm gedruckt. Wir können das per Spracheingabe oder der Alexa-App testen.

* Per Spracheingabe - sagen Sie _Alexa, schalten Sie das Licht ein_ - wenn dies gelingt, antwortet Alexa möglicherweise mit „OK“ oder einem anderen Bestätigungston
* Über die Alexa-App - öffnen Sie Ihre Alexa-Handy-App (nicht die ESP Alexa-Handy-App), gehen Sie zu Geräte und dann entweder **Leuchten** oder **Alle Geräte** und Sie sollten das Gerät mit dem Namen **Light** (siehe Abbildung oben) oder **Demo Light** sehen. Tippen Sie auf das Power-Symbol und das Symbol sollte zwischen Aus und Ein umgeschaltet werden.

Bei beiden Optionen sollte in Ihrem Terminal eine Meldung wie die folgende angezeigt werden:
```bash
I (97445) [alexa_smart_home]: Namespace: Alexa.PowerController, Name: TurnOn
```

In ähnlicher Weise können Sie versuchen, den Helligkeitsbereich mit einer der folgenden Optionen zu steuern:

* Per Spracheingabe - sagen Sie _Alexa, stellen Sie die Helligkeit des Lichts auf 80_ - wenn dies gelingt, antwortet Alexa möglicherweise mit „OK“ oder einer anderen Bestätigungsantwort.
* Passen Sie über die Alexa-App den Helligkeitsregler an.

Bei beiden Optionen sollte im seriellen Monitor eine Meldung wie die folgende angezeigt werden:
```bash
[app_smart_home]: *************** Light's Brightness changed to 80 ***************
```

Dies ist nicht nur nützlich, weil wir mit Alexa einen Sprachassistenten auf unserem Gerät haben, sondern wir können Alexa verwenden, um die Eigenschaften des Geräts selbst zu steuern!

Weiter zum Erstellen eines [**Benutzerdefiniertes Smart Home-Gerät**](/de/intro-to-alexa-for-iot/custom-smart-home-device.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}