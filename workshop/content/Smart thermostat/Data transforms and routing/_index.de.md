+++
title = "Datentransformationen und Routing"
weight = 40
pre = "<b>d. </b>"
+++

## Kapiteleinleitung
Am Ende dieses Kapitels sollte Ihre Cloud-Lösung Folgendes können:

* Umwandlung der vom Gerät empfangenen Schall-Rohdaten in verwertbare Informationen zur Raumbelegung mit in der Cloud gespeicherter Anwendungslogik
* Synchronisieren Sie den Zustand der Raumbelegung vom Geräteschatten bis zum Gerät
* Richten Sie eine Ressource in AWS IoT Events ein, um Nachrichten von Ihrem Gerät zu empfangen
* Weiterleitung der vom Thermostatgerät veröffentlichten Nachrichten an AWS IoT Events zur weiteren Verarbeitung

## So richten Sie die Cloud-Lösung ein
### Raumbelegung aus Umgebungsgeräuschen ableiten
Der erste Teil Ihrer Cloud-Lösung besteht darin, die Intelligenz hinzuzufügen, die ableitet, ob der Raum durch Personen belegt ist. Um die Belegung des Raums zu schätzen, verwenden Sie den vom intelligenten Thermostatgerät gemeldeten Schallpegel. Wenn der abgetastete Schallpegel über einem bestimmten Schwellenwert liegt, markieren Sie den Raum als belegt. Liegt er unter dem Schwellenwert, markieren Sie den Raum als unbelegt. Sie können diesen Status im Geräteschatten speichern und diesen verwenden, um Änderungen zurück zum Gerät zu synchronisieren.

Warum Schallpegel und einfache Schwellenwerte zur Klassifizierung der Raumbelegung verwenden? Warum nicht einen Bewegungssensor verwenden? In diesem Fall ist ein Mikrofon der Sensor, der für den Einsatz zur Verfügung steht. Beim Entwurf von IoT-Lösungen werden Sie nicht immer das Budget für die bestmöglichen Eingangsdaten haben. Dieser Ansatz schafft eine Balance zwischen Sparsamkeit und dem Erreichen des Anwendungsfalls. Der Autor räumt ein, dass dies nicht für jeden Anwendungsfall funktionieren würde, z. B. für eine Teambesprechung für Menschen, die schwerhörig sind und Gebärdensprache verwenden, oder für Teambesprechungen, die mit einer stillen Dokumentenprüfung beginnen.

Sie werden zwei Konzepte von AWS IoT Core verwenden, um diesen Anwendungsfall der Ableitung der Raumbelegung zu erreichen. Dies sind "Topic Rules" und "Device Shadow" (Geräteschatten). Mit einer Topic-Regel können Sie das Verhalten für Nachrichten definieren, die auf dem Topic-Filter ankommen, z. B. die Durchführung von Live-Transformationen an der Payload und das Routing von Payloads an neue Ziele. Der Geräteschatten ist ein semi-strukturiertes JSON-Dokument zur Synchronisierung des gemeldeten und gewünschten Zustands eines Geräts. Alle Änderungen, die Sie am Geräteschatten Ihres Thermostats vornehmen, werden als neue Payloads an Ihr Gerät gesendet. Dass Ihr Gerät diese Shadow-Updates empfängt, haben Sie bereits im vorherigen Kapitel eingerichtet und getestet.

Sie können Themenregeln und Geräteschatten kombinieren, um Updates des Geräteschattens basierend auf der in der Cloud gespeicherten Anwendungslogik funktional durchzuführen. Dies ermöglicht eine schnelle Aktualisierung der Anwendungslogik, ohne neuen Code auf Ihr Gerät zu übertragen.

Ihr erster Meilenstein in diesem Kapitel besteht darin, eine IoT Core-Themenregel zu erstellen, die die von Ihrem intelligenten Thermostat veröffentlichten Nachrichten empfängt, den gemessenen Schallpegel untersucht und den Raumbelegungsstatus Ihres Geräteschattens aktualisiert, wenn er sich ändert. Die Themenregel wird bedingte Logik in der SQL-Abfrage verwenden, um eine neue Payload zu konstruieren, und die IoT Core Republish-Aktion, um die neue Payload an den Geräteschatten zu senden.

1. Gehen Sie zur Konsole von AWS IoT Core, wählen Sie *Act*, wählen Sie *Rules* und wählen Sie **Create**.
2. Geben Sie Ihrer Regel einen Namen und eine Beschreibung. Die weiteren Schritte in diesem Kapitel gehen davon aus, dass der Name `thermostatRule` lautet.
3. Verwenden Sie die folgende Abfrage und stellen Sie sicher, dass Sie **<<CLIENT_ID>>** durch Ihre Geräte-Client-ID/Seriennummer ersetzen.
```SQL
SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
4. Wählen Sie **Add action**.
5. Wählen Sie *Republish a message to an AWS IoT topic* und wählen Sie **Configure action**.
6. Für **Topic** verwenden Sie `$$aws/things/<<CLIENT_ID>>/shadow/update`. Achten Sie darauf,  **<<CLIENT_ID>>** durch die Client-ID/Seriennummer Ihres Geräts zu ersetzen.
7. Für *Choose or create a role to grant AWS IoT access to perform this action* wählen Sie **Create role** und geben Sie im Pop-up Ihrer neuen IAM-Rolle einen Namen und wählen Sie dann **Create role**.
8. Wählen Sie **Add action**, um die Konfiguration Ihrer Aktion abzuschließen und zum Formular für die Regelerstellung zurückzukehren.
9. Klicken Sie auf **Create rule**, um diese Regel in der AWS IoT Rules Engine zu erstellen.

Lassen Sie uns diese Regel aufschlüsseln und die einzelnen Teile erklären. Die SELECT-Klausel verwendet eine CASE-Anweisung, um unser einfaches Schwellenwert-Banding zu erreichen. Wenn der vom Gerät gemeldete Schallpegel über 10 liegt (auf einer Skala von 0-255), dann behandeln wir dies als einen belegten Raum. Sie können die 10 ändern, um den Schwellenwert Ihrer Lösung für einen belegten Raum basierend auf dem beobachteten Grad des Umgebungslärms festzulegen.

Die Ausgabe der CASE-Anweisung wird im Payload-Schlüssel `state.desired.roomOccupancy` mit dem Schlüsselwort AS gespeichert. Das heißt, wir erstellen eine neue Payload wie `{"state": {"desired": {"roomOccupancy": false}}}` und senden diesen Payload an die Aktion weiter.

Die FROM-Klausel beschreibt den Topic-Filter, über den diese Regel neue Nachrichten erhalten soll. In diesem Fall wollen wir Maßnahmen ergreifen, wenn der Geräteschattendienst eine neue Änderung akzeptiert hat, also verwenden wir den Topic-Filter `$aws/things/<<CLIENT_ID>>/shadow/update/accepted`. Es gibt keine Möglichkeit, die vom Gerät auf `$aws/things/<<CLIENT_ID>>/shadow/update` veröffentlichte Nachricht abzufangen und sie während der Übertragung zu ändern, bevor sie an den Geräteschattendienst geht, da die Rule Engine und der Geräteschattendienst Kopien der veröffentlichten Nachricht parallel empfangen. Dieses Verhalten ist genau wie bei zwei anderen Abonnenten des gleichen Topics, wenn eine Nachricht veröffentlicht wird; sie erhalten jeweils eine Kopie. Der Nachteil dieses Musters ist, dass für jeden vom Gerät gemeldeten Schatten eine zweite Schattenaktualisierung veröffentlicht wird, die die roomOccupancy berechnet, wodurch der Datenverkehr für den Schatten effektiv verdoppelt wird. (Eine Möglichkeit, dies zu optimieren, besteht darin, keine neue Schattenmeldung für roomOccupancy zu veröffentlichen, wenn der neue Wert der gleiche ist wie der vorherige. Dies kann durch Ändern der WHERE-Klausel erreicht werden. Diese pädagogische Lösung zieht die Einfachheit der Optimierung vor.)

Die WHERE-Klausel definiert eine bedingte Anweisung, so dass die Aktionen der Regel nur ausgeführt werden, wenn diese Anweisung wahr auflöst. Wir verwenden sie hier, um eine Endlosschleife zu verhindern. Indem wir die bedingte Anweisung `state.reported.sound <> Null` einschließen, konfigurieren wir diese Regel so, dass sie nur auf Shadow-Updates reagiert, bei denen der Sound-Wert enthalten ist (oder besser gesagt, nicht Null). Der Thermostat meldet einen `state.reported.sound`-Wert in der Schattenaktualisierung, so dass diese Regel aktiv wird, wenn der Thermostat Nachrichten veröffentlicht. Wenn die Regel ihr eigenes Schatten-Update veröffentlicht, wird die Regel nicht mehr ausgelöst, da die neue Payload des Schatten-Updates nur den Schlüssel `state.desired.roomOccupancy` und keinen Wert für `state.reported.sound` enthält.

Die Aktion dieser Regel ist "republish", d. h., die Ausgabe der Regelabfrage wird als neue Nachricht in einem angegebenen Topic veröffentlicht. Das Veröffentlichungs-Topic, das wir in dieser Aktion verwenden, ist `$$aws/things/<<CLIENT_ID>>/shadow/update`, um eine neue Nachricht an den Geräteschatten zu senden. Dies ist das gleiche Topic, das Ihr Thermostat über die Geräteschatten-Schnittstelle des C-SDK veröffentlicht.

(Optionale Tiefe) Warum verwenden wir hier zwei $-Zeichen, aber ein einzelnes $ in anderen Verwendungen der Geräteschatten-Topics? Der Grund ist, dass die Rules-Engine-Action optional Substitutionsvorlagen unterstützt. Mit Substitutionsvorlagen können Sie einen Ausdruck definieren, der zur Laufzeit ausgewertet wird. Substitutionsvorlagen verwenden die Notation `${ YOUR_EXPRESSION_HERE }` und dies steht im Konflikt mit dem Präfix `$aws` des Geräteschatten-Topics. Um das richtige Geräteschatten-Topic in einer Wiederveröffentlichungsaktion zu verwenden, müssen Sie das erste $-Zeichen escapen und dies sieht aus wie `$$aws/things/<<CLIENT_ID>>/shadow/update`.

Sobald Sie diese Regel in IoT Core implementiert haben, sollten Sie sehen, dass der Raumbelegungsstatus auf Ihrem Smart-Thermostat-Logger aktualisiert wird (`idf.py monitor -p <<DEVICE_PORT>>`).

### Vorbereitung zur Festlegung von Befehlen für HLK
Der nächste Meilenstein in diesem Kapitel ist die Vorbereitung der Cloud-Infrastruktur, die benötigt wird, um neue HLK-Zustände (z. B. Heizen/Kühlen/Standby) basierend auf der aktuellen Temperatur und Raumbelegung zu diktieren. Sie werden den IoT-Events-Dienst für den Empfang von Nachrichten bereitstellen und dann eine zweite IoT-Core-Regel zur Integration mit IoT-Events erstellen. Dadurch wird der Datenfluss von IoT Core zu IoT Events erstellt, der erforderlich ist, bevor Sie mit der Erstellung des Detektormodells fortfahren, das den HLK-Zustand vorgibt.

IoT Events hat zwei Ressourcentypen: Eingänge (Inputs) und Detektormodelle. Ein Eingang ist ein vordefiniertes Schema zum Abbilden eingehender Nachrichten auf Detektormodelle. Ein Detektormodell ist ein endlicher Zustandsautomat, der Nachrichten von einem oder mehreren Eingängen verarbeitet und bestimmt, ob sich der Zustand des Modells ändern soll.

Gehen Sie folgendermaßen vor, um die Eingangsressource in IoT Events zu erstellen:

1. Gehen Sie zur [AWS Konsole für IoT Events](https://us-west-2.console.aws.amazon.com/iotevents/home?region=us-west-2). Erweitern Sie das linke Menü, wählen Sie **Inputs** und dann **Create input**.
   {{< img "iot_events-create_input.webp" "Choose test in AWS IoT console" >}}
2. Benennen Sie Ihren Input `thermostat` und geben Sie ihm eine Beschreibung. Die weiteren Schritte in diesem Modul beziehen sich auf die Benennung `thermostat`.
3. Sie müssen eine JSON-Datei hochladen, um das Schema zu definieren. Erstellen Sie eine neue Datei auf Ihrem Computer mit folgendem Inhalt und einem Dateinamen wie `input.json`:
```JSON
{
  "current": {
    "state": {
      "reported": {
        "sound": 10,
        "temperature": 35,
        "roomOccupancy": false,
        "hvacStatus": "HEATING"
      }
    },
    "version": 13
  },
  "timestamp": 1606282489
}
```
4. Wählen Sie **Upload file** und wählen Sie die neu erstellte Datei aus. Sie sollten eine Vorschau des aus der Datei interpretierten Schemas sehen.
5. Wählen Sie **Create**, um das Anlegen des Inputs abzuschließen.

Jetzt, da die Eingangsressource in IoT Events erstellt ist, können Sie zu IoT Core zurückkehren, um eine neue Regel zu erstellen, die Geräteschatten-Updates an sie weiterleitet, die Sie zur Erstellung der HLK-Steuerungsanwendung verwenden werden. Sie werden die Steuerungsanwendung im nächsten Kapitel erstellen.

1. Gehen Sie zurück zur [AWS IoT Core Management Konsole](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/) wählen Sie *Act*, wählen Sie *Rules* und wählen Sie **Create**.
2. Geben Sie Ihrer Regel einen Namen und eine Beschreibung.
3. Verwenden Sie diese Anweisung für die Abfrage und stellen Sie sicher, dass Sie **<<CLIENT_ID>>** durch die Client-ID/Seriennummer Ihres Geräts ersetzen:
```SQL
SELECT current.state as current.state, current.version as current.version, timestamp FROM '$aws/things/CLIENT_ID/shadow/update/documents'
```
4. Wählen Sie **Add action**.
5. Wählen Sie *Send a message to an IoT Events Input* und wählen Sie **Configure action**.
6. Suchen Sie für  *Input name* den Namen der Input Resource, die Sie im vorherigen Schritt der IoT-Events Konsole erstellt haben. Der angenommene Name ist `thermostat`.
7. Wählen Sie für *Role* die Option **Create Role**, geben Sie Ihrer Rolle einen Namen und wählen Sie dann **Create Role**, um Ihre neue IAM-Rolle fertigzustellen, die IoT Core die Berechtigung erteilt, Nachrichten an IoT Events zu senden.
8. Wählen Sie **Add action**, um Ihre neue Regelaktion abzuschließen. Sie sollten automatisch zum Formular für die Regelerstellung zurückgeleitet werden.
9. Wählen Sie **Create rule** , um das Erstellen der Regel abzuschließen.

Diese Regel ist im Vergleich zur vorherigen viel einfacher! Die Regel ist so konfiguriert, dass sie das vollständige JSON-Dokument empfängt, wenn der Smart-Thermostat-Geräteschatten aktualisiert wird, und es dann an Ihren neuen IoT-Ereignis-Eingang weiterleitet. Der Eingang ist so konfiguriert, dass er nur einige der Felder aus dem Geräteschatten-Dokument analysiert und die nicht benötigten Felder verwirft.

## Validierungsschritte
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie folgendermaßen überprüfen, ob Ihre Lösung wie vorgesehen konfiguriert ist:

1. Da Ihr Thermostatgerät unterschiedliche Geräuschpegel erkennt, sollten Sie sehen, dass das Gerät einen aktualisierten Status von roomOccupancy von Ihrer Regel erhält. Versuchen Sie, abwechselnd zehn Sekunden lang Musik abzuspielen, um Geräusche zu erzeugen, und zehn Sekunden lang still zu sein, um die Statusänderung im Gerätelogger zu sehen (`idf.py monitor -p <<DEVICE_PORT>>`).

Wenn dies wie erwartet funktioniert, lassen Sie uns mit der [**Cloud-Anwendung**](/de/smart-thermostat/cloud-application.html) fortfahren.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}