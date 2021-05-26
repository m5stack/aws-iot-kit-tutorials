+++
title = "使用机器学习模型"
weight = 40
pre = "<b>d. </b>"
+++

## 章节简介
学完本章节后，您的无服务器应用程序应该能够：
* 使用机器学习模型将新的恒温器消息转化为 roomOccupancy 推理结果。

## 如何设置无服务器基础设施
以下步骤将引导您完成在 AWS Lambda 中创建无服务器函数的流程。此函数定义了一小段代码，用于接收来自 IoT Core 的设备影子消息，将消息转换为机器学习终端节点使用的格式，然后调用机器学习终端节点返回 *roomOccupancy* 分类结果和推理置信度分数。

1. 在 AWS Lambda 控制台中，选择 **Create function（创建函数）**。
2. 为函数命名。后续步骤假定其名称为 `classifyRoomOccupancy`。
3. 在 *Runtime（运行时）* 下，选择 *Python 3.8*。
4. 选择 **Create function（创建函数）**。
5. 在 *Function code（函数代码）* 下的 *lambda_function.py* 文件中，复制并粘贴以下代码来替换占位符代码：
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
6. 在 *Environment variables（环境变量）* 下，选择 **Edit（编辑）**。
7. 对于 *Key（密钥）*，输入 `SAGEMAKER_ENDPOINT`；对于 *Value（值）*，输入 SageMaker 终端节点的名称。在上一章的最后一个步骤您已将此资源命名，因此本模块假定其名称为 `roomOccupancyEndpoint`。
8. 选择 **Save（保存）**，以提交这一新环境变量，并返回 Lambda 编辑器主界面。
9. 在 *Designer（设计器）* 面板中，选择 *+ Add trigger（+ 添加触发器）*。
10. 对于 *Trigger configuration（触发器配置）*，选择列表中的 *AWS IoT*。
11. 对于 *IoT type（IoT 类型）*，选择 *Custom IoT rule（自定义 IoT 规则）*。
12. 对于 *Rule（规则）*，在列表中找到处理来自恒温器的设备影子消息并发布包含 *roomOccupancy* 值的新消息的规则。在上一个模块智能恒温器中，我们假定此规则的名称为 `thermostatRule`。
13. 验证是否已勾选 *Enable trigger（启用触发器）* 复选框，然后选择 **Add（添加）**。此操作将授予 IoT Core 规则调用此 Lambda 函数的权限。
14. 选择 *Permissions（权限）* 选项卡，然后选择 *Role name（角色名称）* 下的链接，为此 Lambda 函数添加调用 SageMaker 终端节点的权限。
15. 在新的选项卡中打开 IAM 控制台，在 *Permissions policies（权限策略）* 下，选择 **Add inline policy（添加内联策略）**。
16. 对于 *Service（服务）*，选择 *SageMaker*。
17. 对于 *Actions（操作）*，选择 *InvokeEndpoint*。
18. 对于 *Resources（资源）*，选择 *All resources（所有资源）*。
19. 选择 **Review policy（审查策略）**。
20. 为您的策略命名，比如将其命名为 `invokeSageMakerEndpoint`，然后选择 **Create policy（创建策略）**。现在，您可以关闭这个新的浏览器选项卡了。

这些步骤总结了 AWS Lambda 函数的配置流程。例如，当 Lambda 函数收到以下设备影子更新时：
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

它将在调用 SageMaker 终端节点后返回以下响应：
```JSON
{
  "statusCode": 200,
  "body": {
    "predicted_label": "false",
    "probability": "0.9999991655349731"
  }
}
```

下一步是更新您的 IoT Core 规则（假定其名称为 `thermostatRule`），以使用此 Lambda 函数进行集成。
1. 返回 IoT Core 控制台，依次选择 **Act（操作）**、**Rules（规则）**，然后选择您的恒温器规则。
2. 选择 *Rule query statement（规则查询语句）* 旁边的 **Edit（编辑）**。它当前应显示为 `SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null`。
3. 将此查询替换为以下新查询：
```SQL
SELECT cast(get(get(aws_lambda("arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME", *), "body"), "predicted_label") AS Boolean) AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
4. 务必要更换占位符 REGION（请检查控制台标头中的当前区域，该区域必须采用 `us-west-2` 这样的格式，而非 `Oregon（俄勒冈）` 这样的格式）、ACCOUNT_ID（在打印您的用户名所在的控制台标头菜单中，使用不带连字符的 12 位数字）以及 FUNCTION_NAME（您所创建的 AWS Lambda 函数的名称，假定其名称为 `classifyRoomOccupancy`）。另外，切勿忘记更新 FROM 主题中的 <<CLIENT_ID>> 占位符。
5. 在 Actions（操作）下，找到名为 *Send a message to a Lambda function（将消息发送到 Lambda 函数）* 的操作，然后选择 **Remove（删除）**。这是您在 Lambda 函数配置中创建触发器以允许此 IoT Core 规则调用此函数时默认添加的，但您不需要此操作。作为替代方式，您将在规则查询中使用内联调用，但仍然需要触发器添加的相同权限。
6. 选择 **Save（保存）**.

此时，您的 IoT 工作流程正在从其部署的终端节点使用经过训练的机器学习模型，以将智能恒温器发布的消息分类为新的 *roomOccupancy* 值！

## 验证步骤
在进入下一章节之前，您可以验证您的无服务器应用程序是否已按预期配置完毕，方法是：

1. 使用 AWS IoT Core 测试客户端，您可以订阅主题 `$aws/things/<<CLIENT_ID>>/shadow/update`（替换 <<CLIENT_ID>>），然后，您将看到两种消息。一种是智能恒温器通过 `state.reported` 发布的的负载。另一种是恒温器规则发布的负载，其中包含机器学习模型确定的 `state.desired.roomOccupancy` 值。

如果没有问题，则说明您已完成本模块的学习，可以进入 [**总结**](/cn/smart-spaces/conclusion.html) 部分了。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}