+++
title = "Benutzerdefiniertes Smart Home-Gerät"
weight = 40
pre = "<b>d. </b>"
+++

## Smart Home-Steuerung anpassen
Nachdem wir die im Kit enthaltenen Smart Home-Steuerungsfunktionen verstanden haben, ändern Sie diese Funktionen, um die Attribute des Geräts selbst zu steuern, anstatt einfach auf dem seriellen Monitor zu drucken. Für diesen Workshop erstellen Sie eine einfache Implementierung, die das seitliche grüne LED-Licht über den **PowerController** einschaltet. Zudem nutzen wir den **RangeController**, um mit einer festgelegten Geschwindigkeit die LEDs blinken zu lassen.

## Anpassen von Smart Home-Geräteattributen
Öffnen Sie mit Ihrer IDE den geklonten Ordner **Core2-for-AWS-IOT-Edukit** und öffnen Sie den `Alexa_for_IOT-Intro/Components/App_Smart_Home/App_Smart_Home.c`, um zu sehen, wo die Smart Home-Geräteattribute definiert sind. Wenn Sie nach unten zur Funktion **app_smart_home_init()** der Zeile 109 scrollen, sehen Sie die folgenden Codeblöcke:
```c
/* Add device */
smart_home_device_t *device = smart_home_device_create("Light", alexa_smart_home_get_device_type_str(LIGHT), NULL);
smart_home_device_add_cb(device, write_cb, NULL);
smart_home_node_add_device(node, device);
```

Der obige Block definiert den „Anzeigenamen“ des Geräts (den Namen des Geräts), die Rückruffunktion **write_cb**, die aufgerufen wird, wenn eine Statusaktualisierungsnachricht für dieses Gerät vorliegt, und das Gerät an den gesamten Smart-Home-Knoten angeschlossen wird. Auf einem Knoten kann sich mehr als ein Smart-Home-Gerät befinden.

Ändern Sie als Nächstes die Definition des Bereichsreglers, um **Brightness** in **Blink** zu ändern. Wenn Sie etwas weiter nach unten zur Zeile 141 scrollen, können Sie sehen, wo die verschiedenen Attribute Ihres Geräts definiert sind:
```c
/* Add device parameters */
smart_home_param_t *power_param = smart_home_param_create("Power", SMART_HOME_PARAM_POWER, smart_home_bool(true), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_device_add_param(device, power_param);

smart_home_param_t *brightness_param = smart_home_param_create("Brightness", SMART_HOME_PARAM_RANGE, smart_home_int(100), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(100), smart_home_int(1));
smart_home_device_add_param(device, brightness_param);
```

Sie müssen die maximale Blinkzahl von `100` auf `10` aktualisieren. Ändern Sie die Variablendefinition **brightness_param** und den Funktionsaufruf **smart_home_param_add_bounds** so, dass sie wie folgt aussehen:

```c
smart_home_param_t *brightness_param = smart_home_param_create("Blink", SMART_HOME_PARAM_RANGE, smart_home_int(10), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(10), smart_home_int(1));
```
## Smart Home-Gerätelogik hinzufügen
Sie haben die Eigenschaften geändert, müssen aber die Logik implementieren, um die grüne LED zu steuern. Zunächst fügen wir eine Bibliothek hinzu, um das Licht zu steuern und einige globale Variablen zu definieren:
```c
#include "core2forAWS.h"
#define BLINK_DELAY 500
static bool green_light_status = 0;
```

Ändern Sie anschließend die Implementierung des Leistungsreglers, um die grüne LED ein- und auszuschalten. Suchen Sie nach der Funktion **write_cb** (Zeile 89 der Originaldatei) und dem folgenden Code-Snippet:

```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");

}
```
Fügen Sie nun eine Umschaltlogik hinzu, die auf den Werten basiert, die Sie durch Hinzufügen von drei Codezeilen erhalten haben, sodass die if-Anweisung wie folgt aussieht:
```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");
    
    //Set the global green_light_status variable to the desired value and set the GPIO1 value the right setting (on/off)
    green_light_status = val.val.b ? 0 : 1;
    Core2ForAWS_LED_Enable(green_light_status);

}
```
Dadurch wird der aktualisierte Wert des Leistungsreglers (1 oder 0) verwendet, um anzuzeigen, ob der Kunde das Gerät ein- oder ausschalten möchte, und schaltet das grüne Seitenlicht ein oder aus.

Implementieren Sie als Nächstes die „Blink“ -Funktionalität. Zunächst erstellen Sie eine Methode, um das grüne Licht eine bestimmte Anzahl von Malen zu blinken. Fügen Sie derselben **app_smart_home.c**-Datei die folgende Funktion hinzu:
```c
static void startLightBlink(int count)
{    
    // Toggle the LED to the opposite of the LED state and get back to the original LED state at the end
    for(int i = (1 + green_light_status); i <= (count * 2 + green_light_status); i++) {
        Core2ForAWS_LED_Enable( i % 2 );
        vTaskDelay(pdMS_TO_TICKS(BLINK_DELAY));
    }
}
```
Aktualisieren Sie nun die RangeController-Logik um diese Funktion aufzurufen. In diesem Fall ist es im folgenden Codeblock wieder in **write_cb**:
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

}
```
Wir müssen unserer neu erstellten **startLightBlink**-Funktion einen Aufruf hinzufügen, also fügen wir hier die folgenden Zeilen hinzu:
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

    //call the Trigger Light Blink based on the desired number of actions
    startLightBlink(val.val.i);        

}
```

## Flashen und Testen der aktualisierten Alexa-Firmware
Sie haben das Projekt so geändert, dass es die erforderlichen Geräteattribute und die Steuerlogik enthält, um die grüne LED an Bord mit einer bestimmten Geschwindigkeit zu blinken. Jetzt ist es an der Zeit, die Firmware zu erstellen, sie auf die Referenzhardware zu flashen und die Funktionen zu testen. Wenn das Gerät angeschlossen und Ihr [PlatformIO CLI Terminalfenster](../blinky-hello-world/prerequisites.html#Öffnen-Sie-das-PlatformIO-CLI-Terminal-Fenster) geöffnet und ausgewählt ist, geben Sie diesen Befehl ein:
```bash
pio run --environment core2foraws --target upload --target monitor 
```
{{% notice info %}}
Sie können den seriellen Monitor mit der Tastenkombination **STRG** + **C** verlassen. Sie können den seriellen Monitor neu starten, indem Sie den Befehl `pio run --environment core2foraws --target monitor ` eingeben.
{{% /notice %}}

Wenn alles gut geht, wird in Ihrer Alexa-App ein Update angezeigt und Sie können die LED-Leistung steuern:
!["Light“ -Gerät hat Benachrichtigung hinzugefügt](custom-smart-home-device/alexa_app-green_light-found.en.jpg?height=500px&classes=shadow)

![Gerät mit dem Namen „Light“ in Ihrer Alexa-App aufgeführt](custom-smart-home-device/alexa_app-green_light-power_on.en.png?height=500px&classes=shadow)

* Stimme: _Alexa, grünes Licht ein-/ausschalten_ - das seitliche grüne LED-Licht sollte aus- und eingeschaltet werden
* Über die Alexa App - öffnen Sie Ihre Alexa-App (nicht die Espressif-App), gehen Sie zu Geräte und dann entweder `Lichter` oder `Alle Geräte` und Sie sollten das Gerät mit dem Namen **Light** sehen (siehe Screenshots unten). Tippen Sie auf das Power-Symbol und das Symbol sollte zwischen Aus und Ein umschalten werden.
  {{< img "alexa_app-green_light-power_press.webp" "Power Controller implemented with the green light" >}}

Sie können auch die Blink-Funktionalität testen:

* Stimme: _Alexa, Blink auf 7 setzen_ - das seitliche grüne LED-Licht sollte 7 mal blinken
* Über die Alexa-App - Stellen Sie vom **Light**-Gerät in Ihrer Alexa-App (nicht in der ESP Alexa-Handy-App) den Schieberegler zwischen 1 und 10 ein und das Gerät sollte die angegebene Anzahl von Malen blinken.
  {{< img "alexa_app-green_light-blink_slider.en.webp" "Blinking the green LED with power controller slider" >}}

Herzlichen Glückwunsch, Sie haben dieses Tutorial abgeschlossen! Weiter zur [**Zusammenfassung**](/de/intro-to-alexa-for-iot/conclusion.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}