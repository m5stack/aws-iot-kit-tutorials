+++
title = "Arbeiten mit ML-Modellen"
weight = 40
pre = "<b>d. </b>"
+++

## Einführung in das Kapitel
Am Ende dieses Kapitels sollte Ihre serverlose Anwendung Folgendes tun:
* Nutzen Sie ein Modell für maschinelles Lernen, das neue Thermostat-Nachrichten in Rückschlüsse auf die Raumbelegung umwandelt.

## So richten Sie die Infrastruktur ohne Server ein
Die folgenden Schritte führen Sie durch die Erstellung einer serverlosen Funktion in AWS Lambda. Die Funktion definiert einen kleinen Code-Ausschnitt, das Geräteschattenmeldungen vom IoT Core erwartet und diese in Nachrichtenfür den ML-Endpunkt umwandelt. Damit rufen Sie Ihren ML-Endpunkt auf, um die Klassifikation von *roomOccupancy* und den Konfidenz-Wert der Inferenz zurückzugeben.

1. Wählen Sie in der AWS Lambda-Konsole **Funktion erstellen**.
2. Geben Sie einen Namen für Ihre Funktion ein. Weitere Schritte gehen von dem Namen `ClassifyRoomOccupancy` aus.
3. Wählen Sie unter *Runtime* *Python 3.8* aus.
4. Wählen Sie **Funktion erstellen**.
5. Kopieren Sie unter *Funktionscode* in die Datei *lambda_function.py* den folgenden Code und ersetzen Sie so den Platzhalter-Code:
```python
import json
import boto3
import os

# Receives a device shadow Accepted document from IoT Core rules engine.
# Event has signature like {"state": {"reported": {"sound": 5}}}.
# See expectedAttributes for full list of attributes expected in state.reported.
# Builds CSV input to send to SageMaker endpoint, name of which stored in
#   environment variable SAGEMAKER_ENDPOINT.
#
# Returns the prediction and confidence score from the ML model endpoint.
def lambda_handler(event, context):
    client = boto3.client('sagemaker-runtime')

    print('event received: {}'.format(event))

    # Order of attributes must match order expected by ML model endpoint. E.g.
    #   the same order of columns used to train the model.
    expectedAttributes = ['sound', 'temperature', 'hvacStatus', 'roomOccupancy', 'timestamp']
    reported = event['state']['reported']
    reported['timestamp'] = event['timestamp']
    reportedAttributes = reported.keys()

    # Validates the input event has all the expected attributes.
    if(len(set(expectedAttributes) & set(reportedAttributes)) < len(expectedAttributes)):
        return {
            'statusCode': 400,
            'body': 'Error: missing attributes from event. Expected: {}. Received: {}.'.format(','.join(expectedAttributes), ','.join(reportedAttributes))
        }

    # Build the input CSV string to send to the ML model endpoint.
    reportedValues = []
    for attr in expectedAttributes:
        reportedValues.append(str(reported[attr]))
    input = ','.join(reportedValues)
    print('sending this input for inference: {}'.format(input))

    endpoint_name = os.environ['SAGEMAKER_ENDPOINT']
    content_type = "text/csv"
    accept = "application/json"
    payload = input
    response = client.invoke_endpoint(
        EndpointName=endpoint_name,
        ContentType=content_type,
        Accept=accept,
        Body=payload
    )

    body = response['Body'].read()

    print('received this response from inference endpoint: {}'.format(body))

    return {
        'statusCode': 200,
        'body': json.loads(body)['predictions'][0]
    }
```
6. Wählen Sie *Konfiguration*, *Umgebungsvariablen* **Bearbeiten**.
7. Für *Schlüssel* geben Sie `SAGEMAKER_ENDPOINT` ein und für *Wert* geben Sie den Namen Ihres SageMaker-Endpoints ein. Sie haben diese Ressource als letzten Schritt des vorherigen Kapitels benannt und dieses Modul geht davon aus, dass der Name `RoomOccupancyEndPoint` ist.
8. Wählen Sie **Speichern**, um diese neue Umgebungsvariable zu übernehmen und zur Lambda-Editor-Hauptoberfläche zurückzukehren.
9. Wählen Sie im *Funktionsüberwicht*-Bedienfeld *+ Auslöser hinzufügen*.
10. Für *Auslöser-Konfiguration* wählen Sie *AWS IoT* aus der Liste.
11. Wählen Sie für *IoT-Typ* *Benutzerdefinierte IoT-Regel*.
12. Suchen Sie für *Regel* Ihre Regel in der Liste, die die Schattenmeldungen des Geräts von Ihrem Thermostat verarbeitet und eine neue Nachricht mit dem Wert *RoomOccupancy* veröffentlicht. Im vorherigen Modul, **Intelligenter Thermostat**, wurde angenommen, dass diese Regel den Namen `thermostatRule` trägt.
13. Vergewissern Sie sich, dass das Kontrollkästchen *Auslöser aktivieren* aktiviert ist, und wählen Sie **Hinzufügen**. Dadurch wird Ihrer IoT-Core-Regel die Berechtigung erteilt, diese Lambda-Funktion aufzurufen.
14. Wählen Sie die Registerkarte *Berechtigungen* unter *Konfiguration* aus und wählen Sie dann den Link unter*Rollenname*, damit Sie Berechtigungen für diese Lambda-Funktion hinzufügen können, um Ihren SageMaker-Endpoint aufzurufen.
15. Wählen Sie auf der neuen Registerkarte, die in der IAM-Konsole geöffnet wurde, unter *Berechtigungsrichtlinie* die Option **Inline-Richtlinie hinzufügen**.
16. Für *Service* wählen Sie *SageMaker*.
17. Für *Actions* wählen Sie *InvokeEndpoint*.
18. Für *Ressourcen* wählen Sie* Alle Ressourcen*.
19. Wählen Sie **Richtlinie überprüfen**.
20. Geben Sie Ihrer Richtlinie einen Namen wie `InvokeSageMakerEndPoint` und wählen Sie **Richtlinie erstellen**. Sie können diesen neuen Browser-Tab jetzt schließen.

Diese Schritte schließen die Konfiguration Ihrer AWS Lambda-Funktion ab. Die Lambda-Funktion erhält nun Device Shadow Updates, wie zum Beispiel:
```JSON
{
  "state": {
    "reported": {
      "sound": 20,
      "temperature": 58.8,
      "hvacStatus": "HEATING",
      "roomOccupancy": true
    }
  },
  "timestamp": 1234567890
}
```

Diese Antwort wird zurückgegeben, nachdem der SageMaker-Endpunkt aufgerufen wurde:

```JSON
{
  "statusCode": 200,
  "body": {
    "predicted_label": "false",
    "probability": "0.9999991655349731"
  }
}
```

Der nächste Schritt besteht darin, Ihre IoT-Core-Regel (angenommener Name von `thermostatRule`) zu aktualisieren, um diese Lambda-Funktionsintegration zu verwenden.
1. Kehren Sie zur IoT-Core-Konsole zurück, wählen Sie **Handeln**, **Regeln** und wählen Sie Ihre Thermostatregel.
2. Wählen Sie **Bearbeiten**. Bein *Regel-Abfrageanweisung* sollte derzeit folgendes stehen: `SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null`.
3. Ersetzen Sie diese Abfrage durch diese neue:
```SQL
SELECT cast(get(get(aws_lambda("arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME", *), "body"), "predicted_label") AS Boolean) AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
4. Achten Sie darauf, die Platzhalter zu ersetzen: Ändern Sie REGION in Ihre aktuelle Region, wie in der Konsolenüberschrift gezeigt (sie muss das Format „us-west-2“ haben und nicht „Oregon“); ändern Sie ACCOUNT_ID in Ihre 12-stellige Konto-ID ohne Bindestriche, die auch im Kopfzeilenmenü der Konsole angezeigt wird, wo Ihr Benutzername gedruckt wird; und ändern Sie FUNCTION_NAME in den Namen der AWS Lambda-Funktion, die Sie erstellt haben (angenommener Name ist `ClassifyRoomOccupancy`). Vergessen Sie nicht, den <<CLIENT_ID>> Platzhalter auch im FROM-Teil zu aktualisieren.
5. Suchen Sie unter Aktionen die Aktion mit dem Namen *Eine Nachricht an eine Lambda-Funktion senden* und wählen Sie **Entfernen**. Dies wurde standardmäßig hinzugefügt, als Sie den Trigger in der Lambda-Funktionskonfiguration erstellt haben, damit diese IoT-Core-Regel ihn aufrufen kann, aber Sie benötigen die Aktion nicht. Stattdessen verwenden Sie in der Regelabfrage einen Inline-Aufruf, benötigen aber immer noch dieselben Berechtigungen, die der Auslöser hinzugefügt hat.
6. Wählen Sie **Speichern**.

Zu diesem Zeitpunkt nutzt Ihr IoT-Workflow Ihr trainiertes Modell für maschinelles Lernen von seinem bereitgestellten Endpunkt aus, um von Ihrem intelligenten Thermostat veröffentlichte Nachrichten als neue *RoomOccupancy*-Werte zu klassifizieren!

## Schritte zur Validierung
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihre serverlose Anwendung wie vorgesehen konfiguriert ist:

1. Verwenden Sie den AWS IoT Core Test Client, um das Thema `$aws/things/<<CLIENT_ID>>/shadow/update` zu abonnieren (ersetzt Ihr <<CLIENT_ID>>) und Sie sollten hier zwei Arten von Nachrichten sehen. Die erste ist die von Ihrem intelligenten Thermostat veröffentlichte Information mit dem Pfad `state.reported`. Die andere ist die Information, die jetzt von Ihrer Thermostatregel mit dem von Ihrem ML-Modell bestimmten Wert `state.desired.roomOccupancy` veröffentlicht wird.

Wenn dies wie erwartet funktionieren, haben Sie dieses Modul abgeschlossen und können mit dem [**Fazit**] (/de/smart-spaces/conclusion.html) fortfahren.

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}