+++
title = "Build und Testen des AFI"
weight = 30
pre = "<b>c. </b>"
+++

## Kapiteleinleitung
In diesem Kapitel werden wir die portierte ESP VA-SDK-Firmware erstellen, sie auf das Gerät flashen, das WLAN bereitstellen und das Gerät für Ihr Alexa-Konto autorisieren sowie einige der Smart-Home-Funktionen mit Alexa-Sprachbefehlen testen, die in dieser Beta-Version von AFI verfügbar sind.

## Flashen der Firmware
Verwenden Sie ESP-IDF, um die Firmware auf das Gerät zu flashen. Ersetzen Sie **<<DEVICE_PORT>>**  durch Ihren Geräteport. Wenn Sie Ihren Geräteanschluss nicht kennen, befolgen Sie die Anweisungen zum [Identifizieren des seriellen Anschlusses auf dem Host-Rechner](/de/blinky-hello-world/device-provisioning.html#identifying-the-serial-port-on-host-machine) im Beispiel **Blinky Hello World** .
```bash
cd path/to/Core2-for-AWS-IoT-EduKit/Alexa_For_IoT-Intro/
idf.py build flash monitor -p <<DEVICE_PORT>>
```

## Bereitstellen des Geräts
Für den Bereitstellungsprozess müssen Sie Ihre Wi-Fi-Netzwerk-Anmeldedaten konfigurieren und die Anwendung mit Ihrem Alexa-Konto über die ESP Alexa Phone Application autorisieren.

Laden Sie die Anwendung aus Ihrem mobilen App-Store herunter.
[iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

Bereitstellungsschritte:

1. Starten Sie die Companion-App.
   - Vergewissern Sie sich, dass Sie Bluetooth aktiviert haben und die App über die richtigen Berechtigungen für den Zugriff auf Bluetooth verfügt.
2. Wählen Sie die Option **Add New Device**.
3. Ihr Gerät wird in der App angezeigt.
   - Sie sehen ein **Proof of Possession**-Fenster mit einem Wert von `abcdf1234`  oder ähnlichem, das erscheint. Drücken Sie einfach **Done**, um fortzufahren.
4. Nachdem Sie sich mit dem Gerät verbunden haben, melden Sie sich bei Ihrem Amazon-Konto an.
5. Wählen Sie das WLAN-Netzwerk aus und geben Sie die Anmeldedaten ein.
6. Nach einer erfolgreichen WLAN-Verbindung sehen Sie eine Liste mit Beispieläußerungen.

## Alexa verwenden
Wenn die vorherigen Schritte abgeschlossen sind, sehen Sie eine Reihe von Logs in Ihrem seriellen Monitor, darunter einige wie die folgenden:

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

Um mit Alexa zu interagieren, müssen Sie *Alexa* zu dem Gerät sagen. Dadurch wird die **Espressif Wake Word Engine**, die auf dem Gerät läuft, in den Aufmerksamkeitszustand **LISTENING** versetzt. Alle Details zu den verschiedenen Aufmerksamkeitszuständen finden Sie in unserer [Dokumentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/ux-design-attention.html#states). Weitere Informationen zur Audioerfassung finden Sie in der [SpeechRecognizer API Dokumentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/avs-speechrecognizer-concepts.html).

{{< img "speechrecognizer-state.en.png" "Audio Capture Speech Recognizer Attention States" >}} 

{{% notice info %}}
Genau wie jedes Alexa-Gerät hört das Gerät im IDLE-Zustand NUR auf das Schlüsselwort "Alexa". Erst wenn das Schlüsselwort ausgelöst wird, beginnt das Gerät mit dem Audio-Streaming in die Cloud.
{{% /notice %}}

Probieren Sie verschiedene Äußerungen an Alexa aus - die seitlichen LEDs sollten blau leuchten, wenn **Alexa** verstanden wird (wenn Alexa nicht "aufwacht", versuchen Sie, näher am Gerät zu sprechen):
* _Alexa, what time is it?_
* _Alexa, tell me a joke?_
* _Alexa, turn on all of the lights_ (Funktioniert nur, wenn Sie bereits Smart-Home Geräte im selben Account haben.)

{{< img "alexa-time.en.webp" "Alexa, what time is it?">}} 

## Testen der Alexa Smart Home-Funktionen (Beta)
Das AFI-Gerät hat **Alexa Built-In**, d.h. Sie können mit Alexa direkt zum Gerät sprechen und Alexa antwortet mit Sprache auf dem Gerät. Diese Version von AFI von Espressif unterstützt jedoch auch Alexa Smart Home-Befehle als Beta-Funktion, mit der Sie Attribute auf dem Gerät steuern können.

Die Alexa for AWS IoT-Beispielanwendung erstellt ein virtuelles Gerät namens **Light** in Ihrer Alexa-App, das zwei Schnittstellen unterstützt:

* [PowerController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-powercontroller.html) um das Licht ein- und auszuschalten.
* [RangeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-rangecontroller.html) um die Helligkeit des Geräts einzustellen.

![The device named "Light" should show up in your Alexa App](building-and-testing-afi/alexa_app-light_Device.en.png?height=500px)

Da es sich um ein virtuelles Gerät handelt, gibt es den aktualisierten Status auf dem Bildschirm aus. Wir können dies per Sprache oder über die Alexa-App testen.

* Per Sprache - sagen Sie _Alexa, turn on the light_ - bei Erfolg antwortet Alexa möglicherweise mit "OK" oder einem anderen Bestätigungston
* Über die Alexa-App - öffnen Sie Ihre Alexa-Mobiltelefon-App (nicht die ESP Alexa-Mobiltelefon-App), gehen Sie zu Geräte und dann entweder **Lampen** oder **Alle Geräte** und Sie sollten das Gerät namens **Lampe** sehen (siehe Screenshots unten). Tippen Sie auf das Stromsymbol und Sie sollten sehen, dass das Symbol zwischen Aus und Ein wechselt.

Durch eine der beiden Optionen sollten Sie eine Meldung wie die folgende in Ihrem Terminal sehen:
```bash
I (97445) [alexa_smart_home]: Namespace: Alexa.PowerController, Name: TurnOn
```

In ähnlicher Weise können Sie versuchen, den Bereich der Helligkeit durch eine der folgenden Möglichkeiten zu steuern:

* Per Sprache - sagen Sie _Alexa, set brightness on the light to 80_ - bei Erfolg antwortet Alexa möglicherweise mit "OK" oder einer anderen Bestätigungsantwort.
* Über die Alexa-App - passen Sie den Schieberegler der Helligkeit an.

Bei beiden Optionen sollten Sie im seriellen Monitor eine Meldung wie die folgende sehen:
```bash
[app_smart_home]: *************** Light's Brightness changed to 60 ***************
```

Dies ist nicht nur nützlich, weil wir mit Alexa einen Sprachassistenten auf unserem Gerät haben, sondern wir können mit Alexa auch Eigenschaften auf dem Gerät selbst steuern!

Weiter zum Erstellen eines [**Benutzerdefinierten Smart Home-Gerätes**](/de/intro-to-alexa-for-iot/custom-smart-home-device.html).


---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}