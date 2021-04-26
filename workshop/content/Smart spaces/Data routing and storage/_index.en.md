+++
title = "Data routing and storage"
weight = 20
pre = "<b>b. </b>"
+++

## Chapter introduction
By the end of this chapter, your serverless application should do the following:

* Forward messages received from the smart thermostat to a managed storage and analytics service
* Be able to run a query against your processed data to produce a materialized view of the results

## Concepts for storing and analyzing IoT data
Up until this point, every aspect of your IoT solution has been ephemeral in the sense that each message is received, processed, and then discarded. In the case of the IoT Events detector model, there is a stateful entity that reacts to new messages, but otherwise there is no stored history of your thermostat messages. This is the step that establishes a data store for your thermostat messages.

AWS offers many ways to store data in the cloud, and storing IoT data is no different. This solution advocates the use of a service called [AWS IoT Analytics](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html), which is a managed service purpose built to receive, store, process, and analyze IoT data in bulk. You will use IoT Analytics to store a subset of each update to the device shadow.

The AWS IoT Analytics documentation covers this in more detail, but here is a brief summary of how it works. The entry point to the service is a resource called a channel. A channel stores all the raw data for your workflow. It also sends a copy of each message received to the next resource, called a pipeline. A pipeline is a series of activities to process, cleanse, filter, and enrich data before it is used in analytics use cases. The processed messages are put from the pipeline into a data store. A data store, like a channel, is a long-lived storage unit for processed data. Finally, a data set is the last resource in the AWS IoT Analytics project. A data set defines a SQL-like query that can read messages out of a data store as a materialized view and deliver the contents of the query to a destination like an S3 bucket.

Your smart space solution will accumulate multiple hours of runtime data from your thermostat in AWS IoT Analytics. After that, the result of a data set query will make this data available for use in our machine learning toolchain provided by Amazon SageMaker.

There is much, much more to [AWS IoT Analytics](https://aws.amazon.com/iot-analytics/) but for the purposes of this module it is the easiest way to store the history of our thermostat messages and aggregate them as a training data set for our machine learning model.

{{% notice warning %}}
Leaving this application running beyond 6 hours can result in AWS charges. It is recommended you finish this tutorial in that time and perform the [cleanup steps](/en/smart-spaces/conclusion.html#clean-up) to avoid any unwanted costs.
{{% /notice %}}

## How to set up the serverless infrastructure
The following steps detail how to create a new IoT Core rule, a new AWS IoT Analytics project, and uses the rule to forward device shadow messages to your AWS IoT Analytics project. There's a handy interface in the IoT Core rule creation wizard for creating the entire AWS IoT Analytics project on your behalf!

1. In the [AWS IoT Core console](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/), choose **Act** then **Rules** then **Create**.
2. Give your rule a name like *storeInIoTAnalytics* and a description.
3. Use the following query. Be sure to replace the **<<CLIENT_ID>>** with the client Id/serial number printed on the screen of your Core2 for AWS IoT Edukit reference hardware kit.

```SQL
SELECT current.state.reported.sound, current.state.reported.temperature, current.state.reported.hvacStatus, current.state.reported.roomOccupancy, timestamp FROM '$aws/things/<<CLIENT_ID>>/shadow/update/documents'
```

4. Choose **Add action** for *Set one or more actions*.
5. Select *Send a message to IoT Analytics* and choose **Configure action**.
6. Select *Quick create IoT Analytics resources* and provide a project name for *Resource prefix*. Further module steps assume the prefix is `smartspace`. Choose **Quick Create** and all your AWS IoT Analytics resources will be created and configured automatically.
7. Choose **Add action** to finish configuring this action and return to the rule creation form. 
8. Choose **Create rule** to finish creating your new rule.

## Validation steps
Before moving on to the next chapter, you can validate that your serverless application is configured as intended by...

1. Ensure that your smart thermostat is powered on, publishing data, and deployed in the room you want to train on.
2. Using the AWS IoT Analytics console, review the most recent data set contents and verify there are historical records of ambient noise levels, temperature, and room occupancy. To check this, find your data set in the [IoT Analytics console](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets), choose **Actions** and **Run now**, then wait for the *Result preview* to update with the latest content. You should see results similar to the following:

{{< img "iot_analytics-dataset_run.en.png" "Running the data set" "1 - Actions, 2 - Run now" >}}
{{< img "iot_analytics-dataset_preview.en.png" "Preview of the data set" >}}

If these are working as expected, let's move on to [**Machine learning**](/en/smart-spaces/machine-learning.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}