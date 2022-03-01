+++
title = "Datensynchronisierung"
weight = 30
pre = "<b>c. </b>"
+++

## Einführung in das Kapitel
Am Ende dieses Kapitels sollte Ihr Core2 des AWS IoT EduKit-Referenzhardware-Kit Folgendes tun:

* Veröffentlichung des abgetasteten Geräuschpegels und der Raumtemperatur alle 10 Sekunden zu AWS IoT Core
* Abonnieren Sie den Schatten Ihres Geräts in AWS IoT Core, um neue Befehle zu erhalten

## Messaging-Konzepte
In diesem Kapitel verbinden Sie Ihr Gerät mit Ihrem benutzerspezifischen AWS IoT Core-Endpunkt und tauschen dann Nachrichten zwischen dem Gerät und der Cloud in einem Modell namens Publish and Subscribe oder „PubSub“ aus. AWS IoT Core basiert stark auf einem Protokoll namens [Message Queueing Telemetry Transport](https://mqtt.org/) (MQTT). Von der Homepage:

> MQTT ist ein OASIS-Standard-Messaging-Protokoll für das Internet der Dinge (IoT). Es ist als extrem leichtgewichtiger Publish/Subscribe-Messaging-Transport konzipiert, der sich ideal für die Verbindung von Remote-Geräten mit geringem Codebedarf und minimaler Netzwerkbandbreite eignet. MQTT wird heute in einer Vielzahl von Branchen wie Automobilindustrie, Fertigung, Telekommunikation, Öl und Gas usw. eingesetzt.

Die Nachrichten, die zwischen Ihrem Gerät und der Cloud ausgetauscht werden, können von Text über Zahlen bis hin zu Binärnachrichten und JSON reichen. JSON ist eine bewährte Methode für den Austausch von Nachrichten zwischen entkoppelten Systemen und ein empfohlener Ansatz für das Arbeiten mit AWS IoT-Services.

Nachrichten werden zu einem Topic veröffentlicht, bei dem es sich um eine Textzeichenfolge handelt, die andere Clients in AWS IoT Core abonnieren können, um Kopien veröffentlichter Nachrichten zu erhalten. Themen können beispielsweise so aussehen:

* `this is a topic`
* `foo/bar`
* `domain/stuff/thing/whatever`
* `dt/device123/temperature`
* `tokyo/weather/report`
* `$aws/things/edukit/shadow/update`

Wie Sie sehen, kann ein Thema ein beliebiger Text sein. Das MQTT-Protokoll definiert Topic-Partitionen, die durch den Schrägstrich '/' getrennt sind. Dies ist ein grundlegendes Konzept für die Gestaltung der Topic-Architektur einer IoT-Lösung. Dieses Modul hilft Ihnen bei der Implementierung von Topics, die den Best Practices entsprechen.

In Ihrer intelligenten Thermostatlösung verwenden Sie eine Funktion von AWS IoT Core, die als Geräteschatten bezeichnet wird. Einfach ausgedrückt ist ein Geräteschatten ein JSON-Dokument, das zum Speichern und Abrufen aktueller Statusinformationen eines Gerätes verwendet wird. Sie können Schlüssel-Wert-Paare in einem JSON-Dokument speichern, das Dokument zu einem spezifischen Topic veröffentlichen, wodurch AWS IoT Core das Dokument in der Cloud speichert. Dies ist eine nützliche Methode, um die neuesten Statusänderungen eines Geräts in der Cloud zu speichern, sodass andere Systeme in Echtzeit Updates erhalten können. Es ist auch für andere Systeme nützlich, die Befehle zurück an Ihr Gerät senden wollen, da AWS IoT Core Ihr Gerät mit allen neuen gewünschten Befehlen synchronisiert, die im Geräteschatten gespeichert sind.

Zum Beispiel muss das intelligente Thermostat die neueste Raumtemperatur und den aktuellen Geräuschpegel melden. Die von Ihrem Gerät veröffentlichte Nachricht sieht folgendermaßen aus:

```JSON
{
  "state": {
    "reported": {
      "temperature": 68,
      "noise": 10
    }
  }
}
```

Wenn AWS IoT Core eine solche Nachricht erhält, speichert es diese Schlüssel-Wert-Paare, bis sie durch einer neu veröffentlichten Nachricht gespeichert werden.

Um zu demonstrieren, wie ein Befehl aussehen könnte, das zurück an Ihr intelligentes Thermostat gesendet wird, könnte beispielsweise diese Nachricht an den Geräteschatten veröffentlicht werden:

```JSON
{
  "state": {
    "desired": {
      "hvacStatus": "COOLING"
    }
  }
}
```

AWS IoT Core führt die JSON-Dokumente so zusammen, dass der aggregierte Status Ihres Geräteschattens wie folgt aussieht:

```JSON
{
  "state": {
    "reported": {
      "temperature": 68,
      "noise": 10
    },
    "desired": {
      "hvacStatus": "COOLING"
    }
  }
}
```

Das heißt, wenn Ihr Gerät das nächste Mal eine Verbindung zu AWS IoT Core herstellt oder wenn es zum Zeitpunkt der Veröffentlichung des gewünschten Befehls bereits verbunden ist, erhält es dieses Aggregatstatus-Dokument. Innerhalb des Gerätecodes müssen Sie festlegeh, auf dieses neue Schlüssel-Wert-Paar `"hvacStatus": "COOLING"` reagiert werden soll. AWS IoT Core macht es einfach, neue Werte zu melden und gewünschte Befehle zu verarbeiten.

Ihr intelligentes Thermostat meldet die neuesten Werte für Temperatur und Geräuschpegel, wie im vorherigen Kapitel beschrieben. Ihr Thermostat empfängt auch Befehle und verfolgt den Status von zwei weiteren Werten, die „hvacStatus“ und „roomOccupancy“ genannt werden. Diese Werte werden in den nächsten Kapiteln cloudseitig festgelegt.

## Wie programmiert man Nachrichten für die Veröffentlichung zu AWS IoT Core
Sie verwenden das AWS IoT Device SDK for Embedded C („C-SDK“), um zwischen Ihrem intelligenten Thermostatgerät und AWS IoT Core zu kommunizieren. Dies ist ein Best Practice für die Abstraktion von Sicherheits-, Netzwerk- und Datenschichten, sodass Sie sich auf die Anwendungslogik Ihres Geräts und Ihrer Lösung konzentrieren können. Das C-SDK bündelt Bibliotheken für die Verbindung mit AWS IoT Core über das MQTT-Protokoll, die Schnittstelle zum sicheren Hardwareelement zum Signieren von Anfragen und zur Integration mit Funktionen höherer Ordnung wie dem Geräteschatten.

Schauen wir uns ein paar kritische Codezeilen an und analysieren, was sie tun.

```C
jsonStruct_t temperatureHandler;
temperatureHandler.cb = NULL;
temperatureHandler.pKey = "temperature";
temperatureHandler.pData = &temperature;
temperatureHandler.type = SHADOW_JSON_FLOAT;
temperatureHandler.dataLength = sizeof(float);
```

Der obige Code definiert eine neue Variable namens _temperatureHandler_ vom Typ `jsonStruct_t`, die wir als Hilfsmittel verwenden, um einzelne Schlüssel-Wert-Paare zu bündeln. Dadurch machen wir die Schlüssel-Wert-Paare für die Verwendung im IoT Core-Geräteschatten verfügbar und bestimmen die zu verwendende Rückruffunktion an (sofern notwendig), falls eine neue „gewünschte“ Nachricht für den angegebenen **pKey** eintreffen sollte. Dieses Beispiel definiert einen neuen Geräteschatten-Schlüssel **temperature**, setzt **pData** auf den Wert der Variablen `temperature` und zeigt an, dass sie vom Typ `SHADOW_JSON_FLOAT` ist. Diese Strukturen werden später verwendet, wenn ein Delta registriert wird und weiterhin die „gemessenen“ Werte zu IoT Core veröffentlicht werden.

```C
rc = aws_iot_shadow_register_delta(&iotCoreClient, &roomOccupancyActuator);
```

Das obige Beispiel registriert das Delta-Verhalten mit dem AWS IoT Device SDK. Es übergibt den Verweis auf den initialisierten SDK-Client und das `jsonStruct_t`, das das Schlüssel-Wert-Paar darstellt, auf dessen Änderungen wir reagieren möchten. Das Beispiel registriert den **roomOccupancyActuator**. Wenn der Client also ein neues Shadow-Update aus der Cloud erhält, das einen neuen Wert für `state.desired.roomOccupancy` enthält, wird die im Aktuator definierte Rückruffunktion verarbeitet.

```C
MPU6886_GetTempData(&temperature);
temperature = (temperature * 1.8)  + 32 - 50;
```

Oben sehen Sie ein Beispiel dafür, wie Sie von der MPU6886-Komponente lesen, um den lokalen Temperaturwert abzurufen. Beachten Sie die Umrechnung in Fahrenheit und den fest codierten Kalibrierungsoffset. Sie können diesen Ausdruck so ändern, dass er mit Celsius oder einem anderen Offset arbeitet, wenn dieser Standardwert von 50 keine Werte im Bereich des erwarteten Bereichs erzeugt.

```C
rc = aws_iot_shadow_add_reported(JsonDocumentBuffer,          
  sizeOfJsonDocumentBuffer, 4, &temperatureHandler,
  &soundHandler, &roomOccupancyActuator, &hvacStatusActuator);
```

Dieser Code packt alle Werte, die wir in der Cloud veröffentlichen möchten, in das Shadow-Dokument, das vom IoT-Core-Schattendienst erwartet wird. Sie können hier Schlüssel-Wert-Paare für `state.reported` hinzufügen oder entfernen, indem Sie diese variadische Funktion ändern. Vergessen Sie nicht, den dritten Parameter so zu aktualisieren, dass er der Anzahl der Schlüssel-Wert-Paare in der Liste entspricht!
```C
rc = aws_iot_shadow_update(&iotCoreClient, 
  CLIENT_ID, JsonDocumentBuffer,
  ShadowUpdateStatusCallback, NULL, 4, true);
```

Dieser Code veröffentlicht das strukturierte Shadow-Dokument an IoT Core zum Thema `$aws/things/<<CLIENT_ID>>/shadow/update`. Die Definition von **<<CLIENT_ID>>** wird in die Client-ID/Seriennummer Ihres Core2 des AWS-Geräts aufgelöst, die auch auf dem Bildschirm Ihres Geräts und in der seriellen Monitorausgabe angezeigt wird.

```C
while(NETWORK_ATTEMPTING_RECONNECT == rc || NETWORK_RECONNECTED == rc || SUCCESS == rc) {
...
  vTaskDelay(1000 / portTICK_RATE_MS);
}
```

Die `while` -Schleife dieses **xTask** wird effektiv für immer ausgeführt, sofern die Netzwerkverbindung nicht unterbrochen wird. Nachdem die neueste Nachricht an den Geräteschatten-Service veröffentlicht und auf Delta-Rückrufe von empfangenen Nachrichten reagiert wurde, wird der Task für die angegebene Zeit im Ruhezustand gehalten, bevor mit dem nächsten Zyklus der Schleife fortgefahren wird. Sie können den `vTaskDelay() `-Ausdruck ändern, um mehr oder weniger häufig zu veröffentlichen. Möglicherweise ist es hilfreich, beim Testen ein schnelleres Intervall wie 1000 (eine Sekunde) zu verwenden und es auf zehn, dreißig oder sechzig Sekunden zu reduzieren.

```C
void hvac_Callback(const char *pJsonString, uint32_t JsonStringDataLen, jsonStruct_t *pContext) {
    IOT_UNUSED(pJsonString);
    IOT_UNUSED(JsonStringDataLen);

    char * status = (char *) (pContext->pData);

    if(pContext != NULL) {
        ESP_LOGI(TAG, "Delta - hvacStatus state changed to %s", status);
    }

    if(strcmp(status, HEATING) == 0) {
        ESP_LOGI(TAG, "setting side LEDs to red");
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0xFF0000);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0xFF0000);
        Core2ForAWS_Sk6812_Show();
    } else if(strcmp(status, COOLING) == 0) {
        ESP_LOGI(TAG, "setting side LEDs to blue");
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0x0000FF);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0x0000FF);
        Core2ForAWS_Sk6812_Show();
    } else {
        ESP_LOGI(TAG, "clearing side LEDs");
        Core2ForAWS_Sk6812_Clear();
        Core2ForAWS_Sk6812_Show();
    }
}
```

Dieser Code stellt eine Rückruffunktion für unseren **hvacStatus**-Aktuator dar. Dies ist der Code, der ausgeführt wird, wenn eine neue Nachricht vom Gerät empfangen wird, die das Schlüssel-Wert-Paar `state.desired.hvacStatus` enthält. Beachten Sie, dass Sie nichts tun müssen, um die gewünschte Statusänderung zu akzeptieren. Sie wird automatisch auf den lokalen **hvacStatusActuator** und dessen **pData** -Element angewendet.

Die `IOTUNUSED()` Funktion wird verwendet, um Compilerwarnungen für nicht verwendete Parameter zu unterdrücken.

Der If/Else-Block wird verwendet, um den Textwert des neuen Schlüsselwerts **HVACStatus** auszuwerten und nutzt diesen, um die Farbe der LED-Streifen zu bestimmen.

## Überwachen der seriellen Geräteausgabe
Wenn Sie den seriellen Monitor aus dem letzten Kapitel geschlossen haben, indem Sie die Tastenkombination **STRG** + **C** gedrückt haben oder das Gerät getrennt haben, müssen Sie den seriellen Monitor neu starten. Führen Sie bei angeschlossenem Gerät den seriellen Monitor aus, indem Sie den folgenden Befehl in das [PlatformIO CLI-Terminalfenster](../blinky-hello-world/prerequisites.html#öffnen-sie-das-platformio-cli-terminal-fenster) einfügen:
```bash
pio run --environment core2foraws --target monitor
```

## Validierungsschritte
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihr Gerät wie beabsichtigt konfiguriert ist:

1. Öffnen Sie die Testseite der AWS IoT Core-Konsole, abonnieren Sie das Thema `$aws/things/<<CLIENT_ID>>/shadow/update/accepted`. Sie sollten neue Nachrichten sehen, die rechtzeitig mit Ihrem **vTaskDelay()** eintreffen. (Ersetzen Sie <<CLIENT_ID>> durch die auf dem Bildschirm aufgedruckte Client-ID/Seriennummer Ihres Geräts.)
2. Öffnen Sie die Testseite der AWS IoT Core-Konsole und veröffentlichen Sie eine neue Shadow-Nachricht zum Thema `$aws/things/<<CLIENT_ID>>/shadow/update`. Sie sollten sehen, dass die LED-Leisten des Core for AWS IoT EduKit von blau über rot zu aus wechseln, um die Werte **COOLING**, **HEATING** und **STANDBY** darzustellen. Unten finden Sie ein Beispiel für eine Shadow-Nachricht. Testen Sie die Auswirkungen, indem Sie bei jeder Veröffentlichung der Nachricht die Werte **hvacStatus** (auf **HEATING** oder **COOLING** eingestellt) und/oder **roomOccupied** (auf **true** oder **false** eingestellt) umschalten.
```
{ "state": { "desired": { "hvacStatus": "HEATING", "roomOccupancy": true } } }
```

Wenn diese Dinge wie erwartet funktionieren, fahren wir mit [**Datentransformationen und Routing**](/de/smart-thermostat/data-transforms-and-routing.html) fort.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}