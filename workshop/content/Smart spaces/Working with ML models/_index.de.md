+++
title = "Arbeiten mit ML-Modellen"
weight = 40
pre = "<b>d. </b>"
+++

## Einleitung
Am Ende dieses Kapitels sollte Ihre serverlose Anwendung Folgendes einsetzen:

- Ein maschinelles Lernmodell, das aus Thermostatinformationen Rückschlüsse auf die Raumbelegung liefert

## So richten Sie die serverlose Infrastruktur ein
The following steps will walk you through creation of a serverless function in AWS Lambda. The function defines a small bit of code that expect device shadow messages from IoT Core, transform the message into the format used with your ML endpoint, then invoke your ML endpoint to return the classification of *roomOccupancy* and the confidence score of the inference.

Die folgenden Schritte führen Sie durch die Erstellung einer serverlosen Funktion in AWS Lambda. Die Funktion definiert ein kleines Stück Code, das Device Shadow-Nachrichten von IoT Core erwartet, die Nachricht in das mit Ihrem ML-Endpunkt verwendete Format umwandelt und dann Ihren ML-Endpunkt aufruft, um die Klassifizierung von *roomOccupancy* und den Konfidenzwert der Inferenz zurückzugeben.

1. Wählen Sie in der AWS Lambda-Konsole **Create function**.
2. Vergeben Sie einen Namen für Ihre Funktion. Die weiteren Schritte übernehmen den Namen `classifyRoomOccupancy`.
3. Wählen Sie unter *Runtime Python 3.8*.
4. Wählen Sie **Create function**.
5. Kopieren Sie unter *Function code* in der Datei *lambda_function.py* folgenden Code und fügen Sie ihn ein, um den Platzhaltercode zu ersetzen:
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
6. Wählen Sie unter *Environment variables* die Option **Edit**.
7. Geben Sie als *Key* `SAGEMAKER_ENDPOINT` und als *Value* den Namen Ihres SageMaker-Endpunkts ein. Sie haben diese Ressource im letzten Schritt des vorherigen Kapitels benannt und dieses Modul nimmt an, dass der Name `roomOccupancyEndpoint` lautet.
8. Wählen Sie **Save**, um diese neue Umgebungsvariable zu übernehmen und zur Hauptoberfläche des Lambda-Editors zurückzukehren.
9. Wählen Sie im *Designer* Feld *+ Add trigger*.
10. Wählen Sie für die *Trigger configuration* **AWS IoT** aus der Liste aus.
11. Wählen Sie für *IoT type* die Option *Custom IoT rule*.
12. Suchen Sie für *Rule* Ihre Regel in der Liste, die die Geräteschattenmeldungen von Ihrem Thermostat verarbeitet und eine neue Meldung mit dem Wert *roomOccupancy* veröffentlicht. Im vorherigen Modul, **Intelligentes Thermostat**, wurde diese Regel als `thermostatRule` angenommen.
13. Überprüfen Sie, ob das Kontrollkästchen *Enable trigger* (Auslöser aktivieren) aktiviert ist, und wählen Sie dann **Add** (Hinzufügen). Dies erteilt Ihrer IoT Core-Regel die Erlaubnis, diese Lambda-Funktion aufzurufen.
14. Wählen Sie die Registerkarte *Permissions* und dann den Link unter *Role name* damit Sie die Berechtigungen für diese Lambda-Funktion zum Aufrufen Ihres SageMaker-Endpunkts hinzufügen können.
15. Wählen Sie in der neu geöffneten Registerkarte der IAM-Konsole unter *Permissions policies* die Option **Add inline policy**.
16. Für *Service* wählen Sie *SageMaker*.
17. Für *Actions* wählen Sie *InvokeEndpoint*.
18. Wählen Sie für *Resources* die Option *All resources*.
19. Wählen Sie **Review policy**.
20. Geben Sie Ihrer Richtlinie einen Namen wie `invokeSageMakerEndpoint` und wählen Sie **Create policy**. Sie können nun diese neue Browser-Registerkarte schließen.

Diese Schritte schließen die Konfiguration Ihrer AWS Lambda-Funktion ab. Wenn die Lambda-Funktion z. B. dieses Geräteschatten-Update empfängt...
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

...gibt sie diese Antwort zurück, nachdem sie den SageMaker-Endpunkt aufgerufen hat...

```JSON
{
  "statusCode": 200,
  "body": {
    "predicted_label": "false",
    "probability": "0.9999991655349731"
  }
}
```

Der nächste Schritt besteht darin, Ihre IoT Core-Regel (angenommener Name `thermostatRule`) zu aktualisieren, um diese Lambda-Funktionsintegration zu verwenden.
1. Return to the IoT Core console, choose **Act**, **Rules**, and choose your thermostat rule.
1. Kehren Sie zur IoT Core-Konsole zurück, wählen Sie **Act**, **Rules** und wählen Sie Ihre Thermostatregel.
2. Wählen Sie **Edit** in der Nähe der *Rule query statement*. Sie sollte wie folgt lauten: 
```SQL   
SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null`.
```
3. Ersetzen Sie diese Abfrage durch diese neue:
```SQL
SELECT cast(get(get(aws_lambda("arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME", *), "body"), "predicted_label") AS Boolean) AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
4. Achten Sie darauf, die Platzhalter REGION (überprüfen Sie Ihre aktuelle Region im Konsolenkopf, sie muss im Format wie `us-west-2` und nicht `Oregon` sein), ACCOUNT\_ID (auch als 12-stellige Zahl, ohne Bindestriche, im Konsolenkopfmenü, wo Ihr Benutzername ausgegeben wird) und FUNCTION_NAME (Name der AWS Lambda-Funktion, die Sie erstellt haben, angenommener Name ist `classifyRoomOccupancy`) zu ersetzen. Vergessen Sie nicht, auch den Platzhalter <<CLIENT_ID>> im Thema FROM zu aktualisieren.
5. Suchen Sie unter Actions die Aktion *Send a message to a Lambda function* und wählen Sie **Remove**. Diese wurde standardmäßig hinzugefügt, als Sie den Auslöser in der Lambda-Funktionskonfiguration erstellt haben, damit diese IoT Core-Regel ihn aufrufen kann, aber Sie brauchen die Aktion nicht. Stattdessen verwenden Sie einen Inline-Aufruf in der Regelabfrage, aber Sie benötigten trotzdem die gleichen Berechtigungen, die der Auslöser hinzugefügt hat.
6. Wählen Sie **Save**.

An diesem Punkt konsumiert Ihr IoT-Workflow nun Ihr trainiertes maschinelles Lernmodell von seinem bereitgestellten Endpunkt, um von Ihrem intelligenten Thermostat veröffentlichte Nachrichten als neue *roomOccupancy* Werte zu klassifizieren!

## Validierungsschritte
Bevor Sie mit dem nächsten Kapitel fortfahren, können Sie überprüfen, ob Ihre serverlose Anwendung wie vorgesehen konfiguriert ist:

1. Mit dem AWS IoT Core Test-Client können Sie das Thema `$aws/things/<<CLIENT_ID>>/shadow/update` abonnieren (ersetzen Sie Ihre <<CLIENT_ID>>) und Sie sollten hier zwei Arten von Nachrichten sehen. Die erste ist die von Ihrem intelligenten Thermostat veröffentlichte Nutzlast mit dem Pfad `state.reported`. Die andere ist die Nutzlast, die jetzt von Ihrer Thermostatregel mit dem Wert `state.desired.roomOccupancy` veröffentlicht wird, der von Ihrem ML-Modell bestimmt wird.

Wenn diese wie erwartet funktionieren, haben Sie dieses Modul abgeschlossen und können mit dem Abschnitt [**Abschluss**](http://localhost:1313/en/smart-spaces/conclusion.html) fortfahren.

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}