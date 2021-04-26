+++
title = "Datensynchronisierung"
weight = 30
pre = "<b>c. </b>"
+++

## Kapiteleinleitung
Am Ende dieses Kapitels sollte Ihr "Core2 for AWS IoT EduKit" Folgendes können:
* Veröffentlichen der gemessenen Umgebungsgeräuschpegel und der Raumtemperatur alle 10 Sekunden an AWS IoT Core
* Abonnieren des Schatten Ihres Geräts in AWS IoT Core, um neue Befehle zu erhalten

## Messaging-Konzepte
In diesem Kapitel verbinden Sie Ihr Gerät mit Ihrem benutzerdefinierten AWS IoT Core-Endpunkt und tauschen dann Nachrichten zwischen dem Gerät und der Cloud in einem Modell aus, das als "Publish and Subscribe" oder "PubSub" bezeichnet wird. AWS IoT Core basiert auf einem Protokoll namens [Message Queueing Telemetry Transport](https://mqtt.org/) (MQTT). Von der Homepage:

> MQTT is an OASIS standard messaging protocol for the Internet of Things (IoT). It is designed as an extremely lightweight publish/subscribe messaging transport that is ideal for connecting remote devices with a small code footprint and minimal network bandwidth. MQTT today is used in a wide variety of industries, such as automotive, manufacturing, telecommunications, oil and gas, etc. 

Die Nachrichten, die zwischen Ihrem Gerät und der Cloud ausgetauscht werden, können alles Mögliche sein, von Text über Zahlen und Binärdaten bis hin zu JSON. JSON ist eine Best Practice für den Austausch von Nachrichten zwischen entkoppelten Systemen und ein empfohlenes Muster für die Erkundung von AWS IoT-Services.

Nachrichten werden in einem Topic (Thema) veröffentlicht. Dabei handelt es sich um eine textuelle Zeichenfolge, die andere Clients auf AWS IoT Core abonnieren können, um Kopien der veröffentlichten Nachrichten zu erhalten. Themen können wie die folgenden Beispiele aussehen:

* `this is a topic`
* `foo/bar`
* `domain/stuff/thing/whatever`
* `dt/device123/temperature`
* `tokyo/weather/report`
* `$aws/things/edukit/shadow/update`


Wie Sie sehen können, kann ein Topic wie ein beliebiger Text aussehen. Das MQTT-Protokoll definiert Topic-Partitionen als durch den Schrägstrich '/' getrennt. Dies ist ein grundlegendes Konzept für den Entwurf einer Topic-Architektur für eine IoT-Lösung. Dieses Modul hilft Ihnen, bei der Implementierung von Topics den Best Practices zu folgen.

In Ihrer Smart-Thermostat-Lösung werden Sie eine Funktion von AWS IoT Core verwenden, die als Geräteschatten bezeichnet wird. Einfach ausgedrückt ist ein Geräteschatten ein JSON-Dokument, das zum Speichern und Abrufen von aktuellen Statusinformationen für ein Gerät verwendet wird. Sie können Schlüssel-Wert-Paare (Key-Value-Pairs) in einem JSON-Dokument speichern, das Dokument in einem speziellen Thema veröffentlichen und AWS IoT Core wird das Dokument in der Cloud speichern. Dies ist eine nützliche Methode, um die neuesten Zustandsänderungen eines Geräts in der Cloud zu speichern, damit andere Systeme Updates in Echtzeit erhalten können. Es ist auch nützlich für andere Systeme, um gewünschte Befehle zurück an Ihr Gerät zu senden, da AWS IoT Core Ihr Gerät mit allen neuen gewünschten Befehlen, die im Geräteschatten gespeichert sind, synchronisiert hält.

Zum Beispiel muss das intelligente Thermostat die aktuelle Raumtemperatur und den Umgebungsgeräuschpegel melden, um unsere Lösung zu betreiben. Die von Ihrem Gerät veröffentlichte Nachricht sieht wie folgt aus:

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

Wenn AWS IoT Core eine solche Nachricht empfängt, speichert es diese Schlüssel-Wert-Paare, bis sie durch eine neue veröffentlichte Nachricht überschrieben werden.

Um zu veranschaulichen, wie ein Befehl an Ihr intelligentes Thermostat zurückgesendet wird, könnte ein anderes System eine Nachricht wie diese in Ihrem Geräteschatten veröffentlichen:

```JSON
{
  "state": {
    "desired": {
      "hvacStatus": "COOLING"
    }
  }
}
```

AWS IoT Core führt die JSON-Dokumente so zusammen, dass der aggregierte Zustand Ihres Geräteschattens wie folgt aussieht:

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

Das bedeutet, dass Ihr Gerät, wenn es sich das nächste Mal mit AWS IoT Core verbindet, oder wenn es zum Zeitpunkt der Veröffentlichung des gewünschten Befehls bereits verbunden ist, dieses aggregierte Zustandsdokument erhalten würde. Es ist Sache des Gerätecodes, auf dieses neue Schlüssel-Wert-Paar `"hvacStatus": "COOLING"` zu reagieren, aber AWS IoT Core macht es einfach, neue Werte zu melden und gewünschte Befehle zu verarbeiten.

Ihr intelligenter Thermostat meldet die aktuellen Werte von Temperatur und Geräuschpegel, wie im vorherigen Kapitel beschrieben. Ihr Thermostat empfängt außerdem Befehle und verfolgt den Zustand von zwei weiteren Werten namens "hvacStatus" und "roomOccupancy". Diese Werte werden in den kommenden Kapiteln von der Cloud-Anwendung ermittelt.

## So programmieren Sie die Veröffentlichung von Nachrichten in AWS IoT Core
Sie verwenden das AWS IoT Device SDK for Embedded C ("C SDK") für die Kommunikation zwischen Ihrem intelligenten Thermostatgerät und AWS IoT Core. Dies ist eine bewährte Methode zur Abstraktion von Sicherheits-, Netzwerk- und Datenebenen, damit Sie sich auf die Anwendungslogik Ihres Geräts und Ihrer Lösung konzentrieren können. Das C-SDK bündelt Bibliotheken für die Verbindung mit AWS IoT Core über das MQTT-Protokoll, die Schnittstelle mit dem Hardware-Sicherheitselement zum Signieren von Anforderungen und für die Integration mit Funktionen höherer Ordnung wie dem Geräteschatten.

Schauen wir uns ein paar kritische Codezeilen an und analysieren wir, was sie tun.

```C
jsonStruct_t temperatureHandler;
temperatureHandler.cb = NULL;
temperatureHandler.pKey = "temperature";
temperatureHandler.pData = &temperature;
temperatureHandler.type = SHADOW_JSON_FLOAT;
temperatureHandler.dataLength = sizeof(float);
```

Der obige Code definiert eine neue `jsonStruct_t`, die wir als Werkzeug zum Packen einzelner Schlüssel-Wert-Paare verwenden, um sie für die Verwendung im IoT Core-Geräteschatten bereit zu machen und die zu verwendende Callback-Funktion anzugeben (falls vorhanden), falls eine neue "gewünschte" Nachricht für den angegebenen **pKey** eintrifft. In diesem Beispiel wird ein neuer Schatten Schlüssel **temperature** definiert, wobei ein anfängliches **pData** auf den Wert der `temperature` Variable gesetzt und angegeben wird, dass es vom Typ `SHADOW_JSON_FLOAT` ist. Diese Strukturen werden später bei der Registrierung des Shadow-Delta-Verhaltens und der Veröffentlichung der "gemeldeten" Werte bis zu IoT Core verwendet.

```C
rc = aws_iot_shadow_register_delta(&iotCoreClient, &roomOccupancyActuator);
```

Das obige Beispiel registriert das Delta-Verhalten mit dem AWS IoT Device SDK. Es übergibt den Verweis auf den initialisierten SDK-Client und die `jsonStruct_t`, die das Schlüssel-Wert-Paar darstellt, auf das wir auf Änderungen reagieren möchten. Das Beispiel registriert den **roomOccupancyActuator**, sodass, wenn der Client ein neues Schatten-Update von der Cloud erhält, das einen neuen Wert für `state.desired.roomOccupancy` enthält, die im Aktor definierte Callback-Funktion verarbeitet wird.

```C
MPU6886_GetTempData(&temperature);
temperature = (temperature * 1.8)  + 32 - 50;
```

Oben sehen Sie ein Beispiel, wie Sie aus dem MPU6886 lesen, um den lokalen Temperaturmesswert zu erhalten. Beachten Sie die Umrechnung in Fahrenheit und den hartkodierten Kalibrierungsoffset. Sie können diesen Ausdruck ändern, um in Celsius oder einem anderen Offset zu arbeiten, wenn dieser Standardwert von 50 keine Werte im Bereich dessen erzeugt, was Sie erwarten.

```C
rc = aws_iot_shadow_add_reported(JsonDocumentBuffer,          
  sizeOfJsonDocumentBuffer, 4, &temperatureHandler,
  &soundHandler, &roomOccupancyActuator, &hvacStatusActuator);
```

Dieser Code packt alle Werte, die wir in der Cloud veröffentlichen möchten, in das Schattendokument, das vom IoT Core-Schatten erwartet wird. Sie können hier Schlüssel-Wert-Paare für `state.reported` hinzufügen oder entfernen, indem Sie diese variadische Funktion ändern. Vergessen Sie nicht, den dritten Parameter zu aktualisieren, damit er mit der Anzahl der Schlüssel-Wert-Paare in der Liste übereinstimmt!

```C
rc = aws_iot_shadow_update(&iotCoreClient, 
  CLIENT_ID, JsonDocumentBuffer,
  ShadowUpdateStatusCallback, NULL, 4, true);
```

Dieser Code veröffentlicht das Schatten-Dokument als Payload über das Netzwerk an IoT Core unter dem Topic `$aws/things/<<CLIENT_ID>>/shadow/update`. Die Definition von **<<CLIENT_ID>>** entspricht der Client-ID/Seriennummer Ihres Core2 for AWS-Geräts, wie sie auf dem Bildschirm Ihres Geräts und in der Ausgabe des Serienmonitors angezeigt wird.

```C
while(NETWORK_ATTEMPTING_RECONNECT == rc || NETWORK_RECONNECTED == rc || SUCCESS == rc) {
...
  vTaskDelay(1000 / portTICK_RATE_MS);
}
```

Die `while`-Schleife dieses **xTask** läuft effektiv ewig, es sei denn, die Netzwerkverbindung wird unterbrochen. Nach dem Veröffentlichen der letzten Nachricht an den Geräteschattendienst und dem Reagieren auf alle Delta-Callbacks von empfangenen Nachrichten schläft die Task für die angegebene Zeit, bevor sie mit dem nächsten Zyklus der Schleife fortfährt. Sie können den Ausdruck `vTaskDelay()` ändern, um mehr oder weniger häufig zu veröffentlichen. Es kann hilfreich sein, beim Testen ein schnelleres Intervall wie 1000 (eine Sekunde) zu verwenden und es für einen echten Einsatz auf zehn, dreißig oder sechzig Sekunden zu reduzieren.

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

Dieser Code stellt eine Callback-Funktion für unseren **hvacStatus**-Aktor dar. Dies ist der Code, der ausgeführt wird, wenn eine neue Nachricht vom Gerät empfangen wird, die das Schlüssel-Werte-Paar `state.desired.hvacStatus` enthält. Beachten Sie, dass Sie nichts tun müssen, um die gewünschte Zustandsänderung zu akzeptieren. Sie wird automatisch auf den lokalen **hvacStatusActuator** und sein **pData**-Element angewandt.

Die Funktion `IOTUNUSED()` dient zur Unterdrückung von Compiler-Warnungen für unbenutzte Parameter.

Im if/else-Block wird der enthaltene Text des neuen **hvacStatus** ausgewertet und damit die Farbe der LED-Streifen bestimmt.

## Code-Beispiel
Sie können das folgende Codebeispiel verwenden, wenn Sie die Anwendung nicht selbst schreiben möchten. Dies wird für diejenigen empfohlen, die mit der Embedded-Entwicklung, FreeRTOS oder der Programmierung eines ESP32 nicht vertraut sind.

1. Wechseln Sie aus dem **Core2-for-AWS-IoT-EduKit** Verzeichnis in das **Smart-Thermostat** Verzeichnis:
   ```bash
   cd Smart-Thermostat
   ```
2. Kompilieren Sie, flashen Sie und starten Sie den seriellen Monitor (ersetzen Sie **<<DEVICE_PORT>>** durch den seriellen Anschluss, der im [**Blinky Hello World**](/de/blinky-hello-world/device-provisioning.html#identifying-the-serial-port-on-host-machine) identifiziert wurde):
   ```bash
   idf.py build flash monitor -p <<DEVICE_PORT>>
   ```
3. Es wird einige Zeit dauern, die App zu erstellen und zu flashen, aber nachdem das erledigt ist, sollten Sie den Strom der Geräteprotokolle in Ihrem Terminal sehen. Sie können die Monitor-Sitzung mit der Tastenkombination **Strg** + **]** schließen.

## Validierungsschritte
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie folgendermaßen überprüfen, ob Ihr Gerät wie vorgesehen konfiguriert ist:

1. Abonnieren Sie auf der Testseite der AWS IoT Core-Konsole das Topic `$aws/things/<<CLIENT_ID>>/shadow/update/accepted` und Sie sollten sehen, dass neue Nachrichten abhängig von Ihrem **vTaskDelay()** eintreffen. (Ersetzen Sie **<<CLIENT_ID>>** durch Ihre Geräte-Client-ID/Seriennummer, die auf dem Bildschirm angezeigt wird.)
2. Veröffentlichen Sie über die Testseite der AWS IoT Core-Konsole eine neue Schattenmeldung zum Topic `$aws/things/<<CLIENT_ID>>/shadow/update`. Sie sollten sehen, dass sich die LED-Balken des Core for AWS IoT EduKit von blau, rot, aus ändern, um die veröffentlichten Werte **COOLING**, **HEATING** und **STANDBY** darzustellen. Siehe unten für eine Beispiel-Schattenmeldung. Testen Sie die Auswirkungen, indem Sie die Werte **hvacStatus** (auf **HEATING** oder **COOLING** eingestellt) und/oder **roomOccupied** (auf **true** oder **false** eingestellt) jedes Mal umschalten, wenn Sie die Nachricht veröffentlichen.

```
{ "state": { "desired": { "hvacStatus": "HEATING", "roomOccupancy": true } } }
```

Wenn alles wie erwartet funktioniert, lassen Sie uns mit [**Datentransformation und Routing**](/de/smart-thermostat/data-transforms-and-routing.html) fortfahren.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}