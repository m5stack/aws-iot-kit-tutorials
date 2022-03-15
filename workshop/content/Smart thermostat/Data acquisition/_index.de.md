+++
title = "Datenerfassung"
weight = 20
pre = "<b>b. </b>"
+++
## Einführung in das Kapitel
Am Ende dieses Kapitels führt Ihr Gerät Folgendes aus:

* Schalten Sie die lokale Smart-Thermostat-Anwendung ein und starten Sie sie.
* Melden Sie den Protokollen die aktuelle gemessene Temperatur, den gemessenen Umgebungsgeräuschpegel, den HLK-Status für Heizung/Kühlung/Standby und eine Anzeige der Raumbelegung.

Das Core2 für AWS IoT EduKit-Referenz-Hardwarekit verfügt über mehrere einsatzbereite Sensoren. Bei dieser Lösung nehmen Sie Messwerte vom Temperatursensor und dem Mikrofon mithilfe des lokalen Anwendungscodes ab. Die Anwendung verfolgt Sensorwerte und Statusmarkierungen, die zum Rendern einer Zusammenfassung auf dem Display verwendet werden.

## Wie programmiere ich die Thermostatanwendung
Ihr intelligenter Thermostat tastet die integrierten Sensoren mit Code ab, der bereits erstellt und in den mitgelieferten Softwarekomponenten enthalten ist. In diesem ersten Schritt erfassen Sie einfach die Werte und drucken sie in den Logger aus, bevor wir Sensorwerte bis zu AWS IoT Core veröffentlichen.

Das Ablesen vom Temperatursensor des mitgelieferten MPU6886-Moduls ist einfach. Hier ist ein Code-Snippet:

```c
// include libraries for interfacing with the kit's modules
#include "freertos/FreeRTOS.h"
#include "core2forAWS.h"
#include "mpu6886.h"
#include "esp_log.h"

// the application to run on the device
void app_main()
{
  // initialize a float to store our temperature reading
  float temperature = 0.0f;

  // initialize the kit modules
  Core2ForAWS_Init();
  MPU6886_Init();

  // read from the temperature sensor
  MPU6886_GetTempData(&temperature);

  // convert the reading to Fahrenheit and apply a calibration offset of -50
  temperature = (temperature * 1.8)  + 32 - 50;

  // write the value to the ESP32 logger
  ESP_LOGI("thermostat", "measured temperature is: %f", temperature);
}
```

Wenn Sie Ihre Geräteprotokolle mit diesem Code erstellen, flashen und überwachen, nimmt das Gerät einen Messwert vom Temperatursensor, schreibt ihn in den Logger und stoppt. Um jede Sekunde kontinuierlich zu testen, ohne andere Prozesse zu blockieren, erstellen Sie eine separate ** [FreeRTOS-Aufgabe](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/system/freertos.html#_CPPv423xTaskCreatePinnedToCore14TaskFunction_tPCKcK8uint32_tPCv11UBaseType_tPC12TaskHandle_tK10BaseType_t) ** und stecken Sie es an einen MCU-Kern an, etwa so:

```c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "core2forAWS.h"
#include "mpu6886.h"
#include "esp_log.h"

// store our application logic in a task function
void temperature_task(void *arg) {
  float temperature = 0.0f;

  // loop forever!
  for (;;) {
    MPU6886_GetTempData(&temperature);
    temperature = (temperature * 1.8)  + 32 - 50;
    ESP_LOGI("thermostat", "measured temperature is: %f", temperature);

    // sleep for 1000ms before continuing the loop
    vTaskDelay(1000 / portTICK_RATE_MS);
  }
}

void app_main()
{
  Core2ForAWS_Init();
  MPU6886_Init();

  // FreeRTOS concept: operations that run in a continuous loop are done in tasks
  xTaskCreatePinnedToCore(&temperature_task, "temperature_task", 4096, NULL, 5, NULL, 1);
}
```

## Kompilieren und Hochladen der Smart Thermostat-Firmware
Es wurde bereits eine Beispielanwendung vorbereitet, mit der Sie mithilfe von Visual Studio Code und der PlatformIO-Erweiterung erstellen und auf Ihr Gerät hochladen können. Wenn Sie bereits ein anderes Projekt in VS-Code geöffnet haben, öffnen Sie zuerst ein neues Fenster (Datei → Neues Fenster), um einen sauberen Datei-Explorer und eine saubere Arbeitsumgebung zu erhalten.

Für dieses Tutorial verwenden Sie das Smart-Thermostat-Projekt. In Ihrem neuen VS-Code-Fenster:
1. Klicken Sie in der VS-Code-Aktivitätsleiste (Menü ganz links) auf das **PlatformIo-Logo**.
2. Wählen Sie im linken PlatformIO-Menü **Öffnen** aus.
3. Klicken Sie auf **Projekt öffnen**.
4. Navigieren Sie zum Ordner `Core2-for-aws-iot-edukit/Smart-Thermostat` und klicken Sie auf **Öffnen**.
   {{< img "pio-home.en.png" "PlatformIO home screen" "1 - Open PIO menu, 2 - Open PIO home, 3 - Open the project folder" >}}

Als Nächstes müssen Sie ein neues PlatformIO CLI-Terminalfenster in VS-Code öffnen:
1) Klicken Sie auf das **PlatformIo-Logo** in der VS-Code-Aktivitätsleiste (Menü ganz links).
2) Wählen Sie im Menü **Schnellzugriff** unter **Verschiedenes** die Option **Neues Terminal**.

{{< img "pio-new_terminal-smart_thermostat.en.png" "PlatformIO CLI terminal in VS Code" "1 - Open PIO menu, 2 - Open new PIO Terminal, 3 - Verify you're in the 'PlatformIO CLI' terminal session, 4 - Paste the commands into terminal, 5 - If you encounter an error autodetecting the port, open the Platform.ini file and follow instructions to manually add the serial port.">}}

Um die Konfiguration aus dem Blinky-Projekt zu kopieren, kompilieren Sie die Gerätefirmware und laden Sie sie auf Ihr Gerät hoch. Befolgen Sie dann die Schritte für das Betriebssystem Ihres Host-Computers, um dieses Kapitel abzuschließen:

{{%expand "Ubuntu or macOS" %}}
1. Kopieren Sie Ihre `sdkconfig`-Datei aus der **Blinky-Hello-World**-Demo, um die Einrichtung Ihres Wi-Fi- und AWS-Endpunkts zu vereinfachen:
   ```bash
   cp ../Blinky-Hello-World/sdkconfig .
   ```
   {{% notice note %}}
   Optional können Sie Ihre Wi-Fi-Anmeldeinformationen und den AWS-Endpunkt für Ihre Firmware manuell festlegen, indem Sie dem Schritt ** [Konfiguration der ESP32-Firmware](/de/blinky-hello-world/connecting-to-aws.html#Konfiguration-der-ESP32-Firmware) ** aus dem Beispiel **Blinky Hello World** folgen.
   {{% /notice %}}
2. Verwenden Sie den folgenden Befehl, um die Firmware zu kompilieren und hochzuladen und die serielle Ausgabe Ihres Geräts zu überwachen. Das Erstellen und Flashen der App dauert einige Zeit, aber danach sollten Sie den Stream der Geräteprotokolle in Ihrem Terminal sehen. Sie können die Monitorsitzung mit der Tastenkombination **Strg** + **C** schließen:
  ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```
{{% /expand%}}
{{%expand "Windows" %}}
1. Kopieren Sie Ihre `sdkconfig`-Datei aus der **Blinky-Hello-World**-Demo, um die Einrichtung Ihres Wi-Fi- und AWS-Endpunkts zu vereinfachen:
   ```bash
   copy ..\Blinky-Hello-World\sdkconfig .
   ```
   {{% notice note %}}
   Optional können Sie Ihre Wi-Fi-Anmeldeinformationen und den AWS-Endpunkt für Ihre Firmware manuell festlegen, indem Sie dem Schritt ** [Konfiguration der ESP32-Firmware](/de/blinky-hello-world/connecting-to-aws.html#Konfiguration-der-ESP32-Firmware) ** aus dem Beispiel **Blinky Hello World** folgen.
   {{% /notice %}}
2. Verwenden Sie den folgenden Befehl, um die Firmware zu kompilieren und hochzuladen und die serielle Ausgabe Ihres Geräts zu überwachen. Das Erstellen und Flashen der App dauert einige Zeit, aber danach sollten Sie den Stream der Geräteprotokolle in Ihrem Terminal sehen. Sie können die Monitorsitzung mit der Tastenkombination **Strg** + **C** schließen:
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```
{{% /expand%}}

## Validierung
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihr Gerät wie vorgesehen konfiguriert ist, indem Sie die serielle Ausgabe im Terminalfenster anzeigen, die wie folgt aussehen sollte:

```
I (16128) MAIN: On Device: roomOccupancy false
I (16132) MAIN: On Device: hvacStatus STANDBY
I (16137) MAIN: On Device: temperature 64.057533
I (16143) MAIN: On Device: sound 8
```

Wenn diese wie erwartet funktionieren, fahren wir mit [**Datensynchronisierung**](/de/smart-thermostat/data-sync.html) fort.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}