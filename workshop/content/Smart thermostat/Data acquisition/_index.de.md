+++
title = "Datenerfassung"
weight = 20
pre = "<b>b. </b>"
+++

## Kapiteleinleitung
Am Ende dieses Kapitels wird Ihr Gerät Folgendes tun:

* Einschalten und Starten der lokalen Smart-Thermostat-Anwendung
* Bericht zu den Protokollen mit der aktuell erfassten Temperatur, dem gemessenen Umgebungsgeräuschpegel, einem HLK-Status von Heizung/Kühlung/Standby und einer Anzeige der Raumbelegung

Das Core2 for AWS IoT EduKit Referenz-Hardware-Kit verfügt über mehrere Sensoren, die Sie verwenden können. Bei dieser Lösung werden Sie mithilfe von lokalem Anwendungscode Messwerte vom Temperatursensor und Mikrofon erfassen. Die Anwendung verfolgt die Sensormesswerte und Statusflags, die zum Rendern einer Zusammenfassung auf dem Display verwendet werden.

## Programmieren der Thermostat Applikation
Ihr intelligentes Thermostat wird die integrierten Sensoren mithilfe der bereits erstellten und in den gebündelten Softwarekomponenten enthaltenen Implementierungen abtasten. In diesem ersten Schritt erfassen Sie einfach die Werte und loggen sie in der Konsole, bevor wir mit der Veröffentlichung der Sensorwerte in AWS IoT Core fortfahren.

Das Auslesen des Temperatursensors des mitgelieferten MPU6886-Moduls ist einfach. Hier ist ein Code-Snippet:

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

Wenn Sie Ihre Geräteprotokolle mit diesem Code erstellen, flashen und monitoren würden, würde das Gerät einen Messwert vom Temperatursensor nehmen, ihn in den Logger schreiben und dann anhalten. Um kontinuierlich jede Sekunde abzutasten, ohne andere Prozesse zu blockieren, sollten Sie einen separaten **[FreeRTOS-Task](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/system/freertos.html#_CPPv423xTaskCreatePinnedToCore14TaskFunction_tPCKcK8uint32_tPCv11UBaseType_tPC12TaskHandle_tK10BaseType_t)** erstellen und ihn an einen MCU-Kern anheften, in etwa so:

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

## Codebeispiel
Es wurde bereits eine Beispielanwendung vorbereitet, die Sie erstellen und auf Ihrem Gerät einsetzen können. Folgen Sie diesen Schritten, um dieses Kapitel abzuschließen.

1. Klonen Sie das Code-Repository für dieses Tutorial, wenn Sie es nicht bereits von einem anderen Tutorial kopiert haben:
   ```bash
   git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
   ```
2. Wechseln Sie in das Verzeichnis des **Smart-Thermostat** Projektes:
   ```bash
   cd Core2ForAWS-AWS-IoT-EduKit/Smart-Thermostat
   ```
3. Kopieren Sie ihre `sdkconfig` Datei von der **Blinky-Hello-World** Demo um die Konfiguration Ihres Wi-Fi und AWS-Endpunkts zu vereinfachen:
   {{%expand "macOS & Linux" %}}
   ```bash
   cp ../Blinky-Hello-World/sdkconfig .
   ```
   {{% /expand%}}
   {{%expand "Windows (64-bit) Command Prompt" %}}
   ```bash
   copy ..\Blinky-Hello-World\sdkconfig .
   ```
   {{% /expand%}}
   {{% notice note %}}
   Sie können optional Ihre Wi-Fi-Anmeldeinformationen und den AWS-Endpunkt für Ihre Firmware manuell einstellen, indem Sie dem Schritt **[Konfigurierung der ESP32 Firmware](/de/blinky-hello-world/connecting-to-aws.html#configuring-the-esp32-firmware)** aus dem **Blinky Hello World**-Beispiel folgen.
   {{% /notice %}}
  
4. Kompilieren Sie, flashen Sie und starten Sie den seriellen Monitor. Ersetzen Sie dabei **<<DEVICE_PORT>>** durch den seriellen Port, an den Ihr Core2 for AWS IoT EduKit-Gerät angeschlossen ist. (Um den seriellen Port zu ermitteln, an den das Gerät angeschlossen ist, lesen Sie bitte [**Blinky Hello World — Identifizieren des seriellen Ports an der Host-Maschine**](/de/blinky-hello-world/device-provisioning.html#identifying-the-serial-port-on-host-machine)):
   ```bash
   idf.py build flash monitor -p <<DEVICE_PORT>> 
   ```
   
5. Es wird einige Zeit dauern, die App zu erstellen und zu flashen, aber nachdem das erledigt ist, sollten Sie den Strom der Geräteprotokolle in Ihrem Terminal sehen. Sie können die Monitor-Sitzung mit der Tastenkombination **Strg** + **]** schließen.

{{% notice note %}}
Wenn Sie an Ihrer Shell-Eingabeaufforderung kein **(edukit)** -Prefix sehen, stellen Sie sicher, dass Sie Ihre conda-Umgebung aktivieren, indem Sie `conda activate edukit` ausführen. Wenn der Befehl idf.py nicht gefunden wird, fügen Sie das ESP-IDF mit dem Befehl `. $HOME/esp/esp-idf/export.sh` (macOS/Linux) oder `%userprofile%\Desktop\esp-idf\export.bat` (Windows) zu Ihrem Pfad hinzu.
{{% /notice %}}

## Validierungsschritte
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihr Gerät wie vorgesehen konfiguriert ist:

1. Wenn Ihr "Core2 for AWS IoT EduKit" eingeschaltet ist und die Anwendung läuft, sollten Sie in Ihrem Terminalfenster Logs sehen können, die folgendermaßen aussehen:

```
I (16128) shadow: On Device: roomOccupancy false
I (16132) shadow: On Device: hvacStatus STANDBY
I (16137) shadow: On Device: temperature 64.057533
I (16143) shadow: On Device: sound 8
```

Wenn diese wie erwartet funktionieren, lassen Sie uns mit der [**Datensynchronisierung**](/de/smart-thermostat/data-sync.html) fortfahren.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}