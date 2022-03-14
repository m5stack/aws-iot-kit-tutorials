+++
title = "Conclusion"
weight = 50
pre = "<b>e. </b>"
+++

## Conclusion
You have completed this hands-on module to upgrade your smart thermostat application to take advantage of a trained machine learning model. You have now previewed the end-to-end experience of how to build an IoT application: acquire physical data from sensors, synchronize state between the edge and the cloud, build a serverless application to control an edge device, and transform data in to actionable insights with machine learning.

If you are looking for ideas on how to extend the solution, here are a few:

* Add a timer to the IoT Analytics data set so that it builds an updated content file once per day.
* Build a simple web UI or provision Amazon QuickSight to visualize the data set created in IoT Analytics.
* Add a new pipeline actitivy in IoT Analytics to fetch the local weather via a Lambda function and add it to your thermostat data.
* Use the Core2 for AWS IoT EduKit reference hardware's libraries to emit a sound whenever the hvacStatus changes.
* Use the Core2 for AWS IoT EduKit reference hardware's libraries to flash the LED strips whenever the roomOccupancy changes.

## Clean up
Between this solution and the previous one (**Smart thermostat**), you created the following resources in AWS:

* IoT Core rules.
* IAM roles.
* IoT Events detector model.
* IoT Analytics project (channel, pipeline, data store, data set).
* Lambda function.
* SageMaker Studio.
* S3 bucket (part of the SageMaker Studio project).
* SageMaker model group.
* SageMaker endpoint.

Your account will continue to accrue metered charges in three ways. First, for provisioned compute resources like the SageMaker endpoint that is hosted on a virtual instance. These resources have hourly charges. Second, for storage of data in resources like S3 buckets and your IoT Analytics project (channel, data store, and processed data set results). These resources have billing dimensions like Gigabyte-month. Third, for event-driven activity like publishing messages to IoT Core, invoking the Lambda function, and processing events in IoT Events, you are billed per event. So if you were to turn off your device, the event-driven activity would cease. You should destroy any of these AWS resources if you are done with the solution and don't intend to continue using them.

The SageMaker endpoint is the most expensive resource as it it has an hourly charge. We recommend that you destroy this resource when you are done using it. Your model itself will persist and could be redeployed later. To destroy the SageMaker endpoint:

1. Go to the SageMaker management console, choose Endpoints, find your endpoint in the list and delete it.

To destroy IoT Analytics storage resources:

1. Go to AWS IoT Analytics, choose Channels, find your channel in the list and delete it.
1. Go to AWS IoT Analytics, choose Data stores, find your data store in the list and delete it.
1. Go to AWS IoT Analytics, choose Data sets, find your data set in the list and delete it.

The IoT Core rule, Lambda function, IoT Events input and detector, and IoT Analytics pipeline only incur further charges as you use them. If you no longer need them, you can destroy each by deleting them from their respective resource detail pages in their respective management consoles:

1. Go to AWS IoT Core, choose Act, choose Rules, find your rules in the list and delete it.
1. Go to AWS IoT Analytics, choose Pipelines, find your pipeline in the list and delete it.
1. Go to AWS Lambda, choose Functions, find your function in the list and delete it.
1. Go to AWS IoT Events, choose Inputs, find your input in the list and delete it.
1. Go to AWS IoT Events, choose Detector models, find your model in the list and delete it.


Go to the next tutorial, [**Intro to Alexa for IoT**](/en/intro-to-alexa-for-iot.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}