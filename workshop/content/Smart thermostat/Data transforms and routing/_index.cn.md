+++
title = "数据转换和路由"
weight = 40
pre = "<b>d. </b>"
+++

## 章节简介
学完本章节后，您的云解决方案应该能够：

* 通过存储在云中的应用程序逻辑，将从设备接收到的原始声音数据转换为房间是否有人的信息
* 将房间占用状态从设备影子同步到设备
* 在 AWS IoT Events 中设置资源以接收来自设备的消息
* 将恒温器设备发布的消息转发到 AWS IoT Events 进行进一步处理

## 如何设置云端解决方案
### 通过声音获取空间占用情况
云解决方案的第一部分是添加空间是否有人占用的信息。要估计空间的占用情况，您可以使用智能恒温器设备报告的声音强度。如果采集的声音强度超过了指定阈值，空间将被标记为有人。如果低于阈值，空间将被标记为无人。您可以将该状态存储在设备影子中，然后用它来将更改同步到设备。

为什么使用声音强度和简单的阈值分类来对空间占用情况进行分类？ 为什么不使用运动传感器？ 在这个例子中，麦克风就是可供使用的传感器。在设计 IoT 解决方案时，您不一定总有预算来获得最好的输入数据。这个方法既节约，又满足了使用场景。当然，我们知道，这种做法不一定适合每个使用场景，例如针对听力障碍人士使用手语的团队会议。

您将使用 AWS IoT Core 的两个概念来实现这个获取空间占用情况的使用案例。这两个概念分别是 *topic rules（主题规则）* 和 *device shadow（设备影子）*。通过主题规则，您可以定义满足主题筛选条件的消息的行为，例如对消息体执行实时转换以及将消息路由到新目的地。设备影子是半结构化 JSON 文档，用于同步 reported 和 desired 的设备状态。对恒温器设备影子的任何修改都将作为新的消息发送到设备。您在之前的章节已经设置并测试了您的设备，确认设备能够接收这些影子更新。

您可以结合主题规则和设备影子，根据存储在云中的应用程序逻辑在功能上更新设备影子。这让您能够快速更新应用程序逻辑，而无需向设备推送新代码。

在本章节中，您的第一个里程碑是创建 IoT Core 主题规则，用于接收智能恒温器发布的消息、检查采集的声音强度以及在设备影子更改时更新空间占用状态。主题规则将使用类似 SQL 查询中的条件逻辑来构建新的消息体，并使用 IoT Core 重新发布操作来向设备影子发送。

1. 登录 AWS IoT Core 管理控制台，依次选择 *Act（操作）*、*Rules（规则）* 和 **Create（创建）**。
2. 输入规则的名称和说明。本材料中的后续步骤假定名称是 `thermostatRule`。
3. 使用下列查询，并确保使用你的设备的客户端 Id/序列号来替换 **<<CLIENT_ID>>**。
```SQL
SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```
4. 选择 **Add action（增加操作）**。
5. 选择 *Republish a message to an AWS IoT topic（重新发布消息到 AWS IoT 主题）*，然后选择 **Configure action（配置操作）**。
6. 对于 *Topic（主题）*, 使用 `$$aws/things/<<CLIENT_ID>>/shadow/update`。确保使用你的设备的客户端 Id/序列号来替换 **<<CLIENT_ID>>**。
7. 对于 *Choose or create a role to grant AWS IoT access to perform this action（选择或创建角色来授予 AWS IoT 执行此操作的权限）。* 选择 **Create Role（创建角色）** 然后在弹出窗口中为新的 IAM 角色命名，再选择 **Create role（创建角色）**.
8. 选择 **Add action（增加操作）** 以完成操作配置并返回规则创建表。
9. 点击 **Create Rule（创建规则）** 在 IoT 规则引擎中创建此规则。

下面来解析这条规则，说明各个部分。SELECT 子句使用了 CASE 语句来实现我们的简单阈值分类。如果设备报告的声音强度超过了 10（范围是 0 到 255），那么我们就将空间视为有人占用。您可以修改 10 的值，根据观测到的环境声音强度设置您的占用空间的解决方案阈值。

CASE 语句与 AS 关键词一起将输出保存到以 `state.desired.roomOccupancy` 为健的消息体中。这意味着我们创建了新的消息，形如 `{"state": {"desired": {"roomOccupancy": false}}}`，并将该消息发送到 action(操作) 中。

FROM 子句描述了这条规则会根据哪个主题筛选条件接收新消息。在这个例子中，我们要在设备影子服务接收新的更改后执行操作，所以我们使用主题筛选条件 `$aws/things/<<CLIENT_ID>>/shadow/update/accepted`。在 `$aws/things/<<CLIENT_ID>>/shadow/update` 上的设备发布的消息发送到设备影子服务之前，您无法拦截该消息，也无法实时修改它，因为规则引擎和设备影子服务同时接收发布的消息的副本。该行为就像订阅同一主题的两个订阅者在消息发布时都会收到副本一样。该模式折中是，设备每报告一个影子，就会发布第二个影子更新，用于计算 roomOccupancy，实际上影子的流量增加了一倍。（一种优化方法是，只要新值与之前的值相同，就不发布新影子消息来计算 roomOccupancy。这种方法可以通过修改 WHERE 子句来实现。这个教程解决方案倾向于简洁的方法而不是优化的方法。）

WHERE 子句定义了条件语句，只有当这个语句解析为真时，才会执行规则的操作。在这里用它可以避免无限循环。通过添加条件语句 `state.reported.sound <> Null`，我们将这条规则配置为：仅针对包含了声音值（或者说声音值非空）的影子更新执行。恒温器会在影子更新中报告 `state.reported.sound` 值，因此这条规则将在恒温器发布消息时执行。当规则发布自己的影子更新时，规则不会再次执行，因为新的影子更新的消息体只包括 `state.desired.roomOccupancy` 键，不包括 `state.reported.sound`。

这条规则的操作是 “重新发布”，或者说是将规则查询的输出作为新的消息发布在指定主题上。我们在此操作中使用的发布主题是 `$$aws/things/<<CLIENT_ID>>/shadow/update` 来将新消息发送到设备影子。恒温器也是在这个主题上通过 C 开发工具包的设备影子接口进行发布的。

（可选扩展内容）为什么我们在这里使用两个 $ 符号，而在其他情况下使用设备影子主题时只有一个 $ 符号？ 这是因为规则引擎操作可以选择支持替换模板。通过替换模板，您可以定义运行时评估的表达式。替换模板使用 `${ YOUR_EXPRESSION_HERE }` 表示法，这与设备影子主题的 `$aws` 前缀冲突。要在重新发布操作中使用正确的设备影子主题，您必须转义第一个 $ 符号，内容类似于 `$$aws/things/<<CLIENT_ID>>/shadow/update`。

在 IoT Core 中部署这条规则后，您应该就能在串行输出 (`pio run --environment core2foraws --target monitor`) 中看到智能恒温器更新后的空间占用状态。

### 为决定 HVAC 的命令做准备
本章节的下一个里程碑是根据当前的温度和空间占用情况对决定 HVAC 新的状态（例如制热/制冷/待机）所需的云基础设施进行准备。您将预置 IoT Events 服务来接收消息，然后创建第二个 IoT Core 规则以与 IoT Events 集成。这将创建从 IoT Core 到 IoT Events 的数据流，以便您创建决定 HVAC 状态的检测器模型。

IoT Events 有两种资源类型：输入和检测器模型。输入是预定义的模式，用于将入站消息映射到检测器模型。检测器模型是有限的状态机，用于处理来自一个或多个输入的消息并确定模型状态是否应该更改。 

按照以下步骤在 IoT Events 中创建输入资源：
1. 登录 [IoT Events 管理控制台](https://us-west-2.console.aws.amazon.com/iotevents/home?region=us-west-2)。展开左侧菜单，选择 **Inputs（输入）**，然后选择 **Create input（创建输入）**。
   {{< img "iot_events-create_input.webp" "Choose test in AWS IoT console" >}}
2. 将输入命名为 `thermostat` 并提供描述说明。本模块的后续步骤需要使用名称 `thermostat`。
3. 您必须上传 JSON 文件以定义模式。使用以下内容和文件名 `input.json` 在您的计算机上创建新文件：
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
4. 选择 **Upload file（上传文件）**，然后选择您创建的新文件。您应该能够看到从文件中解析的模式预览。
5. 选择 **Create（创建）** 以完成输入创建。

在 IoT Events 中创建输入资源后，您可以返回 IoT Core 创建一条新规则，用于将设备影子更新转发到输入资源，您将使用这条规则创建 HVAC 控制应用程序。您将在下一个章节中创建控制应用程序。
1. 回到 [AWS IoT Core 管理控制台](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/)，依次选择 *Act（操作）*、*Rules（规则）* 和 **Create（创建）**。
2. 输入规则的名称和说明。
3. 使用下列查询语句，并确保使用设备的客户端 Id/序列号来替换 **<<CLIENT_ID>>**。
```SQL
SELECT current.state as current.state, current.version as current.version, timestamp FROM '$aws/things/CLIENT_ID/shadow/update/documents'
```
4. 选择 **Add action（添加操作）**。
5. 选择 *Send a message to an IoT Events Input（将消息发送到 IoT Events 输入）*，然后选择 **Configure action（配置操作）**。
6. 对于 *Input name（输入名称）*，找到您之前在 IoT Events 控制台创建的输入资源名称。假定的名称为 `thermostat`。
7. 对于 *Role（角色）*，选择 **Create Role（创建角色）**，为角色命名，然后选择 **Create role（创建角色）**，确定新的 IAM 角色允许 IoT Core 向 IoT Events 发送消息。
8. 选择 **Add action（添加操作）** 来确定新规则操作。您将收到规则创建表格。
9. 选择 **Create rule（创建规则）** 以完成规则创建。

这条规则相比之前的规则简单得多！该规则配置为：在智能恒温器设备影子更新时接收整个 JSON 文档，然后将其转发到新 IoT Events 输入。该输入配置为：仅解析来自设备影子文档的几个字段，并丢弃不需要的字段。

## 验证步骤
在进入下一章节之前，您可以验证您的解决方案是否已按预期配置完毕，方法是：

1. 当恒温器设备检测到变化的声音强度时，您应该可以看到设备从规则接收到更新的 `roomOccupancy` 的状态。尝试播放十秒的音乐来制造声响，然后保持十秒的安静，交替做这两件事并查看串行监视器 (`pio run --environment core2foraws --target monitor`) 中的状态变化。

如果没有问题，我们就进入 [**云应用程序**](/cn/smart-thermostat/cloud-application.html) 部分。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}