+++
title = "Datentransformationen und Routing"
weight = 40
pre = "<b>d. </b>"
+++

## Einführung in das Kapitel
Am Ende dieses Kapitels sollte Ihre Cloud-Lösung Folgendes tun:

* Verwandeln Sie vom Gerät empfangene Roh-Tondaten in verwertbare Informationen zur Raumbelegung mit der Anwendungslogik in der Cloud
* Synchronisieren Sie den Status der Raumbelegung vom Geräteschatten bis zum Gerät
* Richten Sie AWS IoT Events ein, um Nachrichten von Ihrem Gerät zu empfangen
* Leiten Sie Nachrichten, die vom Thermostat veröffentlicht wurden, zur Verarbeitung an AWS IoT Events weiter

## So richten Sie die Cloud-Lösung ein
### Raumbelegung aus Geräuschpegel ableiten
Der erste Teil Ihrer Cloud-Lösung besteht darin, die Intelligenz hinzuzufügen, die bestimmt, ob der Raum belegt ist. Wenn der abgetastete Schallpegel über einem bestimmten Schwellenwert liegt, markieren Sie den Raum als belegt. Wenn es unter dem Schwellenwert liegt, markieren Sie den Raum als nicht belegt. Sie können diesen Status im Geräteschatten speichern und damit Änderungen wieder mit dem Gerät synchronisieren.

Warum sollten Sie den Geräuschpegel und das einfache Schwellwert-System verwenden, um die Raumbelegung zu ermitteln? In diesem Fall ist ein Mikrofon der Sensor, der zur Verfügung steht. Bei der Entwicklung von IoT-Lösungen haben Sie nicht immer das Budget für den bestmöglichen Input. Dieser Ansatz sorgt für ein ausgewogenes Verhältnis zwischen Sparsamkeit und Erfüllen der Zieldefinition. Dies würde zugegebenermaßen nicht immer funktionieren: Bei einer Teambesprechung für Personen, die Gebärdensprache verwenden, zum Beispiel, oder für Teambesprechungen, die mit einer stillen Dokumentenprüfung beginnen.

Sie verwenden zwei Konzepte von AWS IoT Core, um die Raumbelegung zu bestimmen. Dies sind *(Topic-)Regeln* und *Geräteschatten*. Mit einer Regel können Sie das Verhalten für Nachrichten definieren, die mit dem Topic-Filter übereinstimmen, z. B. das Anpassen des Nachrichteninhalts und das Weiterleiten von geänderten Nachricht an neue Ziele. Der Geräteschatten ist ein halbstrukturiertes JSON-Dokument, das verwendet wird, um den gemeldeten und gewünschten Status eines Geräts zu synchronisieren. Alle Änderungen am Geräteschatten Ihres Thermostats werden als neue Nachricht an Ihr Gerät gesendet. Im vorherigen Kapitel haben Sie bereits Ihr Gerät eingerichtet und getestet, ob es diese Shadow-Updates erhält.

Sie können Topic-Regeln und Geräteschatten kombinieren, um den Geräteschatten basierend auf der in der Cloud gespeicherten Anwendungslogik zu aktualisieren. Auf diese Weise können Sie die Anwendungslogik schnell aktualisieren, ohne neuen Code auf Ihr Gerät zu übertragen.

Ihr erster Meilenstein in diesem Kapitel besteht darin, eine IoT-Core-Regel zu erstellen, die die von Ihrem intelligenten Thermostat veröffentlichten Meldungen empfängt, den gesampelten Schallpegel überprüft und den Raumbelegungsstatus Ihres Geräteschattens aktualisiert, wenn sich dieser ändert. Die Regel verwendet bedingte Logik in der SQL-Abfrage, um den neuen Nachrichteninhalt zu erstellen, und die IoT-Core-Aktion zur Wieder-Veröffentlichung zu verwenden, um die neue Nachricht an den Geräteschatten zu senden.

1. Gehen Sie zur AWS IoT Core-Konsole, wählen Sie *Handeln*, wählen Sie *Regeln* und wählen Sie **Erstellen**.
2. Geben Sie ihrer Regel einen Namen und eine Beschreibung. Die folgenden Schritte in diesem Kapitel gehen davon aus, dass der Name `thermostatRule` ist.
3. Verwenden Sie die folgende Abfrage und ersetzen Sie unbedingt **<<CLIENT_ID>>** durch die Client-ID/Seriennummer Ihres Geräts.
```SQL
SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
4. Wählen Sie **Aktion hinzufügen**.
5. Wählen Sie *Nachrichten erneut an ein AWS IoT-Thema veröffentlichen* und wählen Sie **Aktion konfigurieren**.
6. Verwenden Sie für *Thema* `$$aws/things/<<CLIENT_ID>>/shadow/update`. Ersetzen Sie unbedingt **<<CLIENT_ID>>** durch die Client-ID/Seriennummer Ihres Geräts.
7. Für*Erstellen oder wählen Sie eine Rolle aus, um AWS IoT den Zugriff zum Ausführen dieser Aktion zu erteilen.* Wählen Sie **Rolle erstellen** und geben Sie im Pop-up Ihrer neuen IAM-Rolle einen Namen und wählen Sie dann **Rolle erstellen**.
8. Wählen Sie **Aktion hinzufügen**, um die Konfiguration Ihrer Aktion abzuschließen und zum Formular zur Regelerstellung zurückzukehren.
9. Klicken Sie auf **Regel erstellen**, um diese Regel in der AWS IoT-Regel-Engine zu erstellen.

Lassen Sie uns diese Regel aufschlüsseln und die Teile erklären. Die SELECT-Klausel verwendet eine CASE-Anweisung, um unser einfaches Schwellenwertband umzusetzen. Wenn der vom Gerät gemeldete Geräuschpegel über 10 liegt (auf einer Skala von 0-255), behandeln wir diesen als belegten Raum. Sie können die 10 ändern, um den Schwellenwert Ihrer Lösung für einen belegten Raum basierend auf dem beobachteten Grad an Umgebungsgeräuschen festzulegen.

Die Ausgabe der CASE-Anweisung wird im Payload-Schlüssel `state.desired.roomOccupancy` mit dem Schlüsselwort AS gespeichert. Das bedeutet, dass wir eine neue Nachricht wie `{"state": {"desired": {"roomOccupancy": false}}}` erstellen und diese an die Aktion weiterleiten.

Die FROM-Klausel beschreibt den Themenfilter, für den diese Regel neue Nachrichten erhält. In diesem Fall möchten wir Maßnahmen ergreifen, wenn der Device Shadow Service eine neue Änderung akzeptiert hat, daher verwenden wir den Themenfilter `$aws/things/<<CLIENT_ID>>/shadow/update/accepted`. Es gibt keine Möglichkeit, die vom Gerät auf `$aws/things/<<CLIENT_ID>>/shadow/update` veröffentlichte Nachricht abzufangen und während der Übertragung zu ändern, bevor sie zum Device Shadow Service geht, da die Regel-Engine und der Dienst für den Geräteschatten Kopien der veröffentlichten Nachricht parallel erhalten. Dieses Verhalten ist dasselbe, wie wenn es zwei Abonnenten für dasselbe Thema gibt, nachdem eine Nachricht veröffentlicht wird. Sie erhalten jeweils eine Kopie. Der Kompromiss dieses Musters besteht darin, dass für jeden vom Gerät gemeldeten Schatten ein zweites Schattenupdate veröffentlicht wird, das die Raumbelegung berechnet und den Datenverkehr auf dem Schatten effektiv verdoppelt. (Eine Möglichkeit, hier zu optimieren, besteht darin, keine neue Shadow-Nachricht für die Belegung zu veröffentlichen, wenn der neue Wert mit dem vorherigen identisch ist. Dies kann erreicht werden, indem die WHERE-Klausel geändert wird. Die hier umgesetzte Lösung bevorzugt Einfachheit gegenüber Optimierung.)

Die WHERE-Klausel definiert eine bedingte Anweisung, sodass die Aktionen der Regel nur ausgeführt werden, wenn diese Anweisung true auflöst. Wir verwenden es hier, um eine Endlosschleife zu verhindern. Indem wir die bedingte Anweisung `state.reported.sound <> Null` einbeziehen, konfigurieren wir diese Regel so, dass sie nur auf Schattenaktualisierungen wirkt, bei denen der Geräuschpegel enthalten ist (beziehungsweise nicht null). Das Thermostat meldet einen „state.reported.sound“ -Wert in der Schattenaktualisierung, sodass diese Regel anschlägt, wenn das Thermostat Meldungen veröffentlicht. Wenn die Regel ihr eigenes Shadow-Update veröffentlicht, wird die Regel nicht erneut ausgelöst, da das neue Shadow-Update nur den Schlüssel "state.desired.roomOccupancy" und keinen Wert für “state.reported.sound“ enthält.

Die Aktion dieser Regel ist „erneut veröffentlichen“ oder mit anderen Worten, veröffentlichen Sie die Ausgabe der Regelabfrage als neue Nachricht zu einem angegebenen Topic. Das Veröffentlichungs-Topic, das wir in dieser Aktion verwenden, ist `$$aws/things/<<CLIENT_ID>>/shadow/update`, um die neue Nachricht an den Geräteschatten zu senden. Dies ist dasselbe Topic, zu dem Ihr Thermostat über die Geräteschattenoberfläche des C-SDK veröffentlicht.

(Optionale Tiefe) Warum verwenden wir hier zwei $ Zeichen, aber ein einzelnes $ in anderen Anwendungen der Device Shadow-Topics? Der Grund dafür ist, dass die Regel-Engine-Aktion optional Substitutionsvorlagen unterstützt. Mit Substitutionsvorlagen können Sie einen Ausdruck definieren, der zur Laufzeit ausgewertet wird. Substitutionsvorlagen verwenden die Notation `${ YOUR_EXPRESSION_HERE }` und dies steht im Konflikt mit dem Präfix `$aws` des Device Shadow-Themas. Um das richtige Device Shadow-Topic in einer Neuveröffentlichungsaktion zu verwenden, müssen Sie das erste $ -Zeichen "escapen" und dies sieht aus wie `$$aws/things/<<CLIENT_ID>>/shadow/update`.

Sobald Sie diese Regel in IoT Core bereitgestellt haben, sollte der Status der Raumbelegung in der seriellen Monitorausgabe aktualisiert werden (`pio run --environment core2foraws --target monitor`).

### Befehle für HLK vorbereiten
Der nächste Meilenstein in diesem Kapitel ist die Vorbereitung der Cloud-Infrastruktur, die erforderlich ist, um neue HLK-Zustände (z. B. Heizen/Kühlen/Standby) basierend auf der aktuellen Temperatur und Raumbelegung zu bestimmen. Sie stellen den IoT-Events-Dienst bereit, um Nachrichten zu empfangen, und erstellen dann eine zweite IoT-Core-Regel zur Integration mit IoT-Ereignissen. Dadurch wird der Datenfluss vom IoT Core zu den IoT-Ereignissen erstellt, der erforderlich ist, bevor Sie mit der Erstellung des Detektor-Modells fortfahren, das den HLK-Zustand bestimmt.

IoT-Events hat zwei Ressourcentypen: Eingaben und Detektor-Modelle. Eine Eingabe ist ein vordefiniertes Schema für die Zuordnung eingehender Nachrichten zu Detektor-Modellen. Ein Detektor-Modell ist ein endlicher Zustandsautomat, der die Nachrichten von einem oder mehreren Eingängen verarbeitet und bestimmt, ob sich der Zustand des Modells ändern soll.

Gehen Sie folgendermaßen vor, um die Eingaberessource in IoT Events zu erstellen
1. Gehen Sie zu [IoT-Events Konsole](https://us-west-2.console.aws.amazon.com/iotevents/home?region=us-west-2). Erweitern Sie das linke Menü, wählen Sie **Eingaben** und wählen Sie dann **Eingabe erstellen**.
   {{< img "iot_events-create_input.webp" "Choose test in AWS IoT console" >}}
2. Nennen Sie Ihren Eingang `thermostat` und geben Sie ihm eine Beschreibung. Weitere Schritte in diesem Modul hängen davon ab, dass der Name `thermostat` ist.
3. Sie müssen eine JSON-Datei hochladen, um das Schema zu definieren. Erstellen Sie auf Ihrem Computer eine neue Datei mit folgendem Inhalt und einem Dateinamen
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
4. Wählen Sie **Datei hochladen** und wählen Sie die von Ihnen erstellte neue Datei aus. Sie sollten eine Vorschau des interpretierten Schemas sehen.
5. Wählen Sie **Erstellen**, um die Eingabe zu erstellen.

Nachdem die Eingaberessource in IoT-Events erstellt wurde, können Sie zu IoT Core zurückkehren, um eine neue Regel zu erstellen. Diese leitet Veränderungen des Geräteschatten an sie weiter, die Sie zum Erstellen der HLK-Steuerungsanwendung verwenden. Die Steuerungsanwendung erstellen Sie im nächsten Kapitel.
1. Gehen Sie zurück zu [AWS IoT Core Management Konsole](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/), wählen Sie *Handeln*, wählen Sie *Regeln* und wählen Sie **Erstellen**.
2. Gib deiner Regel einen Namen und eine Beschreibung.
3. Verwenden Sie diese Anweisung für die Abfrage und ersetzen Sie unbedingt **<<CLIENT_ID>>** durch die Client-ID/Seriennummer Ihres Geräts:
```SQL
SELECT current.state as current.state, current.version as current.version, timestamp FROM '$aws/things/CLIENT_ID/shadow/update/documents'
```
4. Wählen Sie **Aktion hinzufügen**.
5. Wählen Sie *Eine Nachricht an einen IoT Events-Eingabe schicken* und wählen Sie **Aktion konfigurieren**.
6. Suchen Sie für *Eingabenamen* den Namen der Eingabe-Ressource, die Sie im vorherigen Schritt der IoT-Events-Konsole erstellt haben. Der Name sollte `thermostat`sein.
7. Wählen Sie für *Rolle* **Rolle erstellen**, geben Sie Ihrer Rolle einen Namen und wählen Sie dann **Rolle erstellen**, um die IoT Core zu berechtigen, Nachrichten an IoT-Events zu senden.
8. Wählen Sie **Aktion hinzufügen** und kehren Sie so zum Formular zur Regelerstellung zurück.
9. Wählen Sie **Regel erstellen**, um die Regel fertig zu erstellen.

Diese Regel ist viel einfacher als die vorherige! Die Regel ist so konfiguriert, dass sie das vollständige JSON-Dokument erhält, wenn der Schatten des intelligenten Thermostats aktualisiert wird, und leitet es dann an Ihre neue IoT Events-Eingabe weiter. Die Eingabe ist so konfiguriert, dass nur einige der Felder aus dem Shadow-Dokument des Geräts analysiert, und die nicht benötigten Felder verworfen werden.

## Schritte zur Validierung
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihre Lösung wie beabsichtigt konfiguriert ist:

1. Da Ihr Thermostat unterschiedliche Geräuschpegel erkennt, sollte das Gerät von Ihrer Regel den aktualisierten Status der Raumbelegung erhalten. Versuchen Sie, abwechselnd Musik abzuspielen, um zehn Sekunden lang Geräusche zu erzeugen, und zehn Sekunden lang leise zu sein, um zu sehen, wie sich der Status auf dem seriellen Monitor ändert (`pio run --environment core2foraws --target monitor`).

Wenn es wie erwartet funktioniert, gehen wir zur [**Cloud-Anwendung**](/de/smart-thermostat/cloud-application.html) über.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}