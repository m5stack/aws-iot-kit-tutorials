+++
title = "Benutzerdefiniertes Smart Home-Gerät"
weight = 40
pre = "<b>d. </b>"
+++

## Smart Home Steuerung anpassen
Jetzt, da wir wissen, welche Smart Home-Steuerungsfunktionen im Kit enthalten sind, werden Sie diese Funktionen modifizieren, um Attribute des Geräts selbst zu steuern, anstatt einfach auf den seriellen Monitor zu drucken. In diesem Workshop werden Sie eine einfache Implementierung erstellen, die das seitliche grüne LED-Licht über den **PowerController** einschaltet, und wir werden das Gerät so einstellen, dass es mit einer bestimmten Rate blinkt, indem wir den Range Controller verwenden.

## Anpassen der Attribute von Smart Home-Geräten
Öffnen Sie mit Ihrer IDE den geklonten Ordner **Core2-for-AWS-IoT-EduKit** und öffnen Sie die `Alexa_for_IoT-Intro/components/app_smart_home/app_smart_home.c`, um einen Blick darauf zu werfen, wo die Smart Home-Geräteattribute definiert sind. Scrollen Sie bis zur Zeile 109 der Funktion **app_smart_home_init()**  und Sie werden die folgenden Codeblöcke sehen:

```c
/* Add device */
smart_home_device_t *device = smart_home_device_create("Light", alexa_smart_home_get_device_type_str(LIGHT), NULL);
smart_home_device_add_cb(device, write_cb, NULL);
smart_home_node_add_device(node, device);
```

Der obige Block definiert den "Friendly Name" des Geräts, d. h. - den Namen des Geräts. Seien Sie deutlicher mit dem Namen, indem Sie **Light** in **Green Light** ändern. Die zweite Zeile sollte wie folgt aussehen:

`smart_home_device_t *device = smart_home_device_create("Green Light", alexa_smart_home_get_device_type_str(LIGHT), NULL);`

Ändern Sie als Nächstes die Definition des Range Controllers, um **Brightness** in **Blink** zu ändern. Wenn Sie etwas weiter nach unten zu Zeile 141 scrollen, können Sie sehen, wo die verschiedenen Attribute unseres Geräts definiert sind:

```c
/* Add device parameters */
smart_home_param_t *power_param = smart_home_param_create("Power", SMART_HOME_PARAM_POWER, smart_home_bool(true), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_device_add_param(device, power_param);

smart_home_param_t *brightness_param = smart_home_param_create("Brightness", SMART_HOME_PARAM_RANGE, smart_home_int(100), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(100), smart_home_int(1));
smart_home_device_add_param(device, brightness_param);
```

Sie müssen die maximale Blinkanzahl von 100 auf 10 aktualisieren. Sie nehmen also eine Änderung an der Definition der Variablen **brightness_param** und am Funktionsaufruf **smart_home_param_add_bounds** vor, so dass es jetzt wie folgt aussehen sollte:

```c
smart_home_param_t *brightness_param = smart_home_param_create("Blink", SMART_HOME_PARAM_RANGE, smart_home_int(10), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(10), smart_home_int(1));
```

## Hinzufügen der Logik für Smart Home-Geräte
Inzwischen haben Sie die Eigenschaften geändert, müssen aber noch die Logik zur Steuerung der grünen LED implementieren. Zunächst müssen wir eine Bibliothek für die Steuerung des Lichts einbinden und auch ein paar globale Variablen definieren:
```c
#include "axp192.h"
static uint8_t CURRENT_BLINK_DELAY = 20;
static uint8_t GREEN_LIGHT_STATUS = 0;
```

Ändern Sie als Nächstes die Implementierung des Leistungscontrollers, um die grüne LED ein-/auszuschalten. Suchen Sie nach der Funktion **write_cb**  (Zeile 89 der Originaldatei) und dem folgenden Code-Snippet:
```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");

}
```

Nun fügen Sie eine Umschaltlogik basierend auf den empfangenen Werten hinzu, indem Sie drei Codezeilen hinzufügen, so dass die if-Anweisung wie folgt aussieht:
```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");
    
    //Set the global GREEN_LIGHT_STATUS variable to the desired value and set the GPIO1 value the right setting (on/off)
    GREEN_LIGHT_STATUS = val.val.b ? 0 : 1;
    Axp192_SetGPIO1Mode(GREEN_LIGHT_STATUS);

}
```

Sie nehmen den aktualisierten Wert des Leistungsreglers (1 oder 0), um anzuzeigen, ob der Kunde das Gerät ein- oder ausschalten wollte, und schalten das seitliche grüne Licht ein oder aus.

Als nächstes implementieren Sie die "Blink"-Funktionalität. Zuerst müssen Sie eine Methode erstellen, um das grüne Licht eine bestimmte Anzahl von Malen blinken zu lassen. Fügen Sie die folgende Funktion in die gleiche Datei **app_smart_home.c** ein:
```c
static void startLightBlink(int count)
{    
    //we want to start toggling based on the opposite status of what the light currently is
    for(int i = 1 + GREEN_LIGHT_STATUS; i <= count*2+GREEN_LIGHT_STATUS ; i++) {               
        Axp192_SetGPIO1Mode( i % 2 );
        vTaskDelay(CURRENT_BLINK_DELAY);
    }
}
```

Aktualisieren Sie nun die RangeController-Logik, um diese Funktion aufzurufen. In diesem Fall ist es im folgenden Code-Block zurück in **write_cb**:
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

}
```

Wir müssen nur einen Aufruf zu unserer neu erstellten *startLinkBlink*-Funktion hinzufügen, also können wir die folgenden Zeilen hier hinzufügen:
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

    //call the Trigger Light Blink based on the desired number of actions
    startLightBlink(val.val.i);        

}
```

## Flashen und Testen der aktualisierten Alexa-Firmware
Nachdem das Projekt so modifiziert wurde, dass es die erforderlichen Geräteattribute und die Steuerlogik für das Blinken der grünen Onboard-LED mit einer bestimmten Rate aufweist, ist es an der Zeit, die Firmware zu erstellen, sie auf die Referenzhardware zu flashen und die Fähigkeiten zu testen. Führen Sie den gleichen Befehl wie in den anderen Tutorials aus, wobei Sie **<<DEVICE_PORT>>** durch den seriellen Port ersetzen, an den Ihr "Core2 for AWS IoT EduKit"-Gerät angeschlossen ist.

```bash
idf.py build flash monitor -p <<DEVICE_PORT>>
```
{{% notice info %}}
Sie können den seriellen Monitor mit der Tastenkombination `STRG` + `]` beenden. Sie können den seriellen Monitor neu starten, indem Sie den Befehl `idf.py monitor -p <<DEVICE_PORT>>` eingeben.
{{% /notice %}}

Wenn alles gut geht, sehen Sie eine Aktualisierung in Ihrer Alexa-App, dass Sie ein neues Gerät namens **Green Light** haben. Versuchen Sie, den Strom zu steuern: 
!["Green Light" Gerät added notification](custom-smart-home-device/alexa_app-green_light-found.en.jpg?height=500px&classes=shadow)

![Gerät namens "Green Light" in Ihrer Alexa App](custom-smart-home-device/AlexaApp-GreenLight.png?height=500px&classes=shadow)

* Sprache: _Alexa, turn on/off green light light_ - das seitliche grüne LED-Licht sollte aus- und eingeschaltet werden.
* Über die Alexa-App - öffnen Sie Ihre Alexa-App (nicht die Espressif-App), gehen Sie zu "Geräte" und dann entweder zu "Lichter" oder zu "Alle Geräte", und Sie sollten das Gerät mit dem Namen "Green Light" sehen (siehe Screenshots unten). Tippen Sie auf das Stromsymbol und Sie sollten sehen, dass das Symbol zwischen Aus und Ein wechselt.

{{< img "alexa_app-green_light-power_press.webp" "Power Controller implemented with the green light" >}}

Sie können auch die Blink-Funktionalität testen:

* Sprache: _Alexa, set blink to 7_ - die seitliche grüne LED-Leuchte sollte 7 Mal blinken.
* Über die Alexa-App - Stellen Sie auf dem **Green Light**-Gerät in Ihrer Alexa-App (nicht in der ESP Alexa-Handy-App) den Schieberegler zwischen 1 und 10 ein und das Gerät sollte die angegebene Anzahl von Malen blinken.
{{< img "alexa_app-green_light-blink_slider.en.webp" "Blinking the green LED with power controller slider" >}}

Herzlichen Glückwunsch, Sie haben dieses Tutorial abgeschlossen! Weiter geht es mit der [**Zusammenfassung**](/de/intro-to-alexa-for-iot/conclusion.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}