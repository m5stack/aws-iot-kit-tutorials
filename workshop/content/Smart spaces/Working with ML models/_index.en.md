+++
title = "Working with ML models"
weight = 40
pre = "<b>d. </b>"
+++

## Introduction
By the end of this chapter, your serverless application should do the following:
* Consume a machine learning model that translates new thermostat messages into inferences of roomOccupancy.

## Set up the serverless infrastructure
The following steps will walk you through creation of a serverless function in AWS Lambda. The function defines a small bit of code that expects device shadow messages from IoT Core, transforms the message into the format used with your ML endpoint, then invokes your ML endpoint to return the classification of *roomOccupancy* and the confidence score of the inference.

1. From the AWS Lambda console, choose **Create function**.
2. Enter a name for your function. Further steps assume the name `classifyRoomOccupancy`.
3. Under *Runtime*, select *Python 3.8*.
4. Choose **Create function**.
5. Under *Function code*, in the file *lambda_function.py*, copy and paste the following code to replace the placeholder code:
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
6. Under *Environment variables*, choose **Edit**.
7. For *Key* enter `SAGEMAKER_ENDPOINT` and for *Value* enter the name of your SageMaker endpoint. You named this resource as the last step of the previous chapter and this module assumes the name is `roomOccupancyEndpoint`.
8. Choose **Save** to commit this new environment variable and return to the main Lambda editor interface.
9. In the *Designer* panel, choose *+ Add trigger*.
10. For *Trigger configuration* select *AWS IoT* from the list.
11. For *IoT type*, select *Custom IoT rule*.
12. For *Rule*, find your rule in the list that processes the device shadow messages from your thermostat and publishes a new message with the *roomOccupancy* value. In the previous module, **Smart thermostat**, this rule was assumed to be named `thermostatRule`.
13. Verify the *Enable trigger* checkbox is enabled, then choose **Add**. This grants permission to your IoT Core rule to invoke this Lambda function.
14. Select the *Permissions* tab, then choose the link under *Role name* so you can add permissions for this Lambda function to invoke your SageMaker endpoint.
15. From the new tab opened to the IAM console, under *Permissions policies* choose **Add inline policy**.
16. For *Service* choose *SageMaker*.
17. For *Actions* choose *InvokeEndpoint*.
18. For *Resources* choose *All resources*.
19. Choose **Review policy**.
20. Give your policy a name like `invokeSageMakerEndpoint` and choose **Create policy**. You can now close this new browser tab.

These steps conclude configuration of your AWS Lambda function. When the Lambda function receives this device shadow update, for example:
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

It will return this response after invoking the SageMaker endpoint:
```JSON
{
  "statusCode": 200,
  "body": {
    "predicted_label": "false",
    "probability": "0.9999991655349731"
  }
}
```

The next step is to update your IoT Core rule (assumed name of `thermostatRule`) to use this Lambda function integration. 
1. Return to the IoT Core console, choose **Act**, **Rules**, and choose your thermostat rule.
2. Choose **Edit** near *Rule query statement*. It should currently read `SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null`.
3. Replace this query with this new one: 
```SQL
SELECT cast(get(get(aws_lambda("arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME", *), "body"), "predicted_label") AS Boolean) AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
4. Be sure to replace the placeholders: change REGION to your current region as shown in the console header (it must be in the format `us-west-2` and not `Oregon`); change ACCOUNT_ID to your 12-digit account id, without hyphens, which is also shown in the console header menu where your username is printed; and change FUNCTION_NAME to the name of the AWS Lambda function you created (assumed name is `classifyRoomOccupancy`). Don't forget to update the <<CLIENT_ID>> placeholder in the FROM topic as well.
5. Under Actions, find the action called *Send a message to a Lambda function* choose **Remove**. This was added by default when you created the trigger in the Lambda function configuration to allow this IoT Core rule to invoke it, but you don't need the action. Instead, you are using an inline invocation in the rule query, but you still need the same permissions that the trigger added.
6. Choose **Save**.

At this point, your IoT workflow is now consuming your trained machine learning model from its deployed endpoint to classify messages published by your smart thermostat as new *roomOccupancy* values! 

## Validation
Before moving on to the next chapter, you can validate that your serverless application is configured as intended:

1. Use the AWS IoT Core Test client to subscribe to the topic `$aws/things/<<CLIENT_ID>>/shadow/update` (replacing your <<CLIENT_ID>>) and you should see two kinds of messages here. The first is the payload published by your smart thermostat with the `state.reported` path. The other is the payload now being published by your thermostat rule with the `state.desired.roomOccupancy` value determined by your ML model.

If these are working as expected, you have completed this module and can move on to [**Conclusion**](/en/smart-spaces/conclusion.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}