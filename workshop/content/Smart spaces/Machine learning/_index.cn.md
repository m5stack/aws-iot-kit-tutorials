+++
title = "机器学习"
weight = 30
pre = "<b>c. </b>"
+++

## 章节简介
学完本章节后，您的无服务器应用程序应该能够：

* 训练机器学习模型，将新的恒温器消息转化为对 roomOccupancy 的推理。
* 将机器学习模型托管在可供使用的 API 终端节点上。

## 训练机器学习模型的概念
数据科学和机器学习本身都是庞大的领域。本模块的内容只是其中的一小部分，介绍了有关机器学习模型训练的基本知识。幸运的是，创建新模型的工具链已经简化，只要我们对数据有足够的了解，就可以利用该工具链来对机器学习进行实验。

在此解决方案中，您要替换 IoT Core 规则引擎中的一个简单阈值，该值用于评估设备报告的声音强度，并强制生成一个新的布尔键值，称为 *roomOccupancy*。通过上报消息中的声音数据可以发现，安静时该值较低，有噪音时较高。所以，像 “大于 10” 等简单的阈值很适合作为生成 *roomOccupancy* 值的初始值。（在特定案例中，其他环境声音阈值可能更有效！）

您将应用类似的方法，具体是：为机器学习工具链提供从部署设备的空间记录的恒温器数据样本，并告知训练作业 “当 roomOccupancy 为真时数据是怎样的，以及当 roomOccupancy 为假时数据是怎样的。” 该训练作业将评估您的数据集、以 *roomOccupancy* 列为目标，并尝试建立一个模型，来根据声音强度范围甚至其他列（例如时间、HVAC 状态和温度），准确再现 *roomOccupancy* 值。

成功训练首个模型后，按照模型的定义，您将在后续章节中使用 roomOccupancy 推理结果的分类来 _替换_ 简单的声音强度静态阈值！

## 如何设置无服务器基础设施
首先，您需要设置 Amazon SageMaker Studio，以便为自动模型训练配置新的实验。

1. 前往 Amazon SageMaker 控制台，然后选择 **Amazon SageMaker Studio**。
2. 选择 *Quick start（快速入门）*，然后可以选择输入一个新的 *User name（用户名）*。
3. 对于 *Execution role（执行角色）*，选择下拉列表并选择 **Create a new IAM role（创建新的 IAM 角色）**。
4. 对于 *S3 buckets you specify（指定的 S3 存储桶）*，选择 **None（无）**，然后选择 **Create role（创建角色）**。名称中带有 “sagemaker” 的存储桶（其他保持默认值）对于这个项目来说已经足够了。
5. 选择 **Submit（提交）** 以启动 SageMaker Studio 预置流程。系统需要几分钟的时间才能代您完成此操作。

SageMaker Studio 预置完成后，下一步是打开 Studio 并配置一个新项目。

1. 在 SageMaker Studio 控制面板中，选择 **Open Studio（打开 Studio）**。
2. 在 Launcher（启动器）选项卡中，选择 **New project（新项目）**。
3. 在 *SageMaker project templates（SageMaker 项目模板）* 中，选择 *MLOps template for model building, training, and deployment（用于模型构建、训练和部署的 MLOps 模板）*，然后选择 **Select project template（选择项目模板）**。
4. 为项目命名并提供描述说明，然后选择 **Create project（创建项目）**。

项目创建完成后，您将可以看到包含 Repositories（存储库）、Pipelines（管道）、和 Experiments（实验）等选项卡的项目控制面板。在 SageMaker Studio 中将此浏览器选项卡保留为打开状态，以便快速返回到此页面。

下一个任务是返回 AWS IoT Analytics，以便导出汇总的恒温器数据，供新的机器学习项目使用。

1. 打开 [AWS IoT Analytics 控制台](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets)，然后选择您的数据集（假设其名称为 `smartspace_dataset`）。
2. 在 *Data set content delivery rules（数据集内容分发规则）* 下，选择 **Edit（编辑）**。
3. 选择 **Add rule（添加规则）**，然后选择 **Deliver result to S3（将结果发送到 S3）**。
4. 在 **S3 bucket（S3 存储桶）** 下，选择 **Please select a resource（请选择资源）**，然后找到为 SageMaker Studio 项目创建的 S3 存储桶。它的名称类似于 `sagemaker-project-p-somehashhere`。如果存在多个以这种形式命名的存储桶，那么您需要查看 SageMaker Studio 项目，获取项目的随机哈希 ID。您可以在项目的其他资源（例如 Repositories（存储库）选项卡和 Pipelines（管道）选项卡）中查看哈希值。
5. 在 *Bucket key expression（存储桶键表达式）* 下，使用以下表达式：`data/smartspace/Version/!{iotanalytics:scheduleTime}_!{iotanalytics:versionId}.csv`
6. 在 *Role（角色）* 下，选择 **Create new（创建新角色）** 并为 IAM 角色命名，该角色将授予 IoT Analytics 访问权限，以将数据写入 S3 存储桶。选择 **Create role（创建角色）**。
7. 选择 **Save（保存）**，以敲定新的分发规则。

{{< img "iot_analytics-dataset_delivery.en.png" "Content delivery rule" >}}

8. 要生成保存到新的 Amazon S3 存储桶的用于训练的数据集，请选择 **Actions（操作）**，然后选择 **Run now（立即运行）**。数据集内容生成后，您将在 *Result preview（结果预览）* 中看到更新的信息。

{{< img "iot_analytics-dataset_run.en.png" "Running the data set""1 - Actions（操作）, 2 - Run now（立即运行）" >}}
{{< img "iot_analytics-dataset_preview.en.png" "Preview of the data set" >}}

现在，您可以返回 SageMaker Studio 开始机器学习实验了。实验将使用上报的恒温器数据（刚刚由 IoT Analytics 数据集导出）作为输入。您将对实验进行配置，以便准确预测当前 “roomOccupancy” 列的数据。自动训练作业将分析您的数据以便尝试使用相关算法，然后使用不同的超参数来运行 250 个训练作业，从而选出最适合输入训练数据的超参数。

开始机器学习实验之前，您需要拥有几个小时的上报数据，这些数据应该来自您想要分析的会议室内的恒温器，并且这个时段内的会议室状态应该同时包括有人状态和无人状态。要进行自动机器学习实验，至少需要 500 行数据，但提供的数据越多，结果越准确。如果在开始实验之前仍然需要生成更多数据，请务必在 IoT Analytics 控制台中再次运行数据集（前面所讲的说明中的最后一步），以便 SageMaker 可以从项目 S3 存储桶中获取结果。当您准备好开始实验时，请按以下步骤操作。

1. 返回 SageMaker Studio，打开您的项目，选择 *Experiments（实验）* 选项卡，然后选择 **Create autopilot experiment（创建自动实验）**。
2. 为实验命名。
3. 在 *Project（项目）* 下，从列表中选项您的项目。
4. 在 *Connect your data（连接数据）* 和 *S3 bucket name（S3 存储桶名称）* 下，从列表中找到并选择项目的 S3 存储桶。选择的存储桶与在上一章中为 IoT Analytics 数据集内容分发规则选择的存储桶相同。
5. 在 *Dataset file name（数据集文件名）* 下，找到并选择 IoT Analytics 数据集内容，类似：`data/smartspace/Version/1607276270943_3b4eb6bb-8533-4ac0-b8fd-1b62ac0020a2.csv`。
6. 在 *Target（目标）* 下，选择 `roomoccupancy`。
7. 在 *Output data location（输出数据位置）* 和 *S3 bucket name（S3 存储桶名称）* 下，从列表中找到并选择您在第 4 步中选择的项目 S3 存储桶。
8. 在 `output/smartspace` 中的 *Dataset directory name（数据集目录名称）* 类型下，选择 **Use input as S3 object key prefix “output/smartspace”（使用输入作为 S3 对象键前缀 “output/smartspace”）**。这将在 S3 存储桶中定义一个新的前缀，用于输出文件。
9. 选择 **Create Experiment（创建实验）**，以开始自动机器学习实验。 

运行实验需要上小时的时间。您可以在 SageMaker Studio 浏览器选项卡中跟踪实验的进度，也可以关闭该选项卡，稍后再打开该选项卡查看进度。

实验结束后，最终输出 250 次试验的结果，SageMaker 根据这些试验结果来确定最合适的优化作业参数。对试验表进行排序，找出标记为 *Best（最准确）* 的试验。下一个里程碑是将该试验部署为模型终端节点，以便您可以将其作为 API 调用。

1. 选择标记为 *Best（最佳）* 的试验，然后选择 **Deploy model（部署模型）**。
   {{< img "sagemaker-trials.png" "SageMaker trials" "1 - Best（最佳）, 2 - Deploy model（部署模型）" >}}
2. 为终端节点命名。本模块中的后续步骤假定其名称为 `roomOccupancyEndpoint`。
3. 在 *Inference Response Content（推理响应内容）* 下，选择 *predicted_label* 和 *probability（概率）*。*predicted_label* 可能已添加到列表中。
4. 选择 **Deploy model（部署模型）**，告知 SageMaker 将模型部署为可使用的新 API 终端节点。这一过程需要几分钟。 

{{< img "sagemaker-deploy.png" "SageMaker deploy" >}}

现在，您的机器学习模型已经部署为 API 终端节点，并由 Amazon SageMaker 管理。在下一章 **使用机器学习模型** 中，您将通过无服务器函数来使用该 API 终端节点，并将 IoT Core 规则中用于确定 `roomOccupancy` 值的简单阈值逻辑替换为模型生成的推理结果。

{{% notice warning %}}
由于对 S3 存储桶进行的请求，此应用程序运行操作 6 个小时后可能会产生 AWS 上的费用。推荐在您完成教程之后执行 [清理步骤](/cn/smart-spaces/conclusion.html#clean-up) 避免成本的支出。
{{% /notice %}}

## 验证步骤
在进入下一章节之前，您可以验证您的无服务器应用程序是否已按预期配置完毕，方法是：
1. 在 [Amazon SageMaker 控制台](https://us-west-2.console.aws.amazon.com/sagemaker/home?region=us-west-2#/endpoints) 中的 *Endpoints（终端节点）* 页面上，您应该能够看到状态为 *InService（服务中）* 的新终端节点。

{{< img "sagemaker-endpoints.png" "SageMaker endpoints" >}}

如果没有问题，我们就进入到 [**使用机器学习模型**](/cn/smart-spaces/working-with-ml-models.html) 部分。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-Kit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}