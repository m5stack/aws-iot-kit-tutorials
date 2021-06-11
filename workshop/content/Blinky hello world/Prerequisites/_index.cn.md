+++
title = "先决条件"
weight = 10
pre = "<b>a. </b>"
+++

在本章节中，您将下载安装适用您的操作系统的 [AWS CLI](https://aws.amazon.com/cli/)，检索 AWS [Identity and Access Management](https://aws.amazon.com/iam/) (IAM) 用户访问凭证（AWS CLI 将使用该凭证管理 AWS 服务），配置 AWS CLI，最后测试 AWS CLI 工作正常。本教程假定您拥有 [AWS 账户](https://console.aws.amazon.com/console/home) 并且已经完成了 [配置您的环境](../getting-started/prerequisites.html)。如果您已经安装和配置了 AWS CLI(version 1 or version 2)，跳过 [测试部分](#testing-the-aws-cli)。 

## 打开 PlatformIO CLI 终端窗口
在 [入门](../getting-started.html) 教程中，你安装和使用了 PIO 和 PIO 终端窗口。在结下来的步骤中，将会继续使用 PIO 终端窗口。PIO 终端窗口预加载了附带的应用程序和库，您的标准的终端/命令接口中一般并没有这些应用程序和库。

如果你已经关闭了 VS Code 或者在 VS Code 中并没有加载 PlatformIO CLI 的终端视口，在打开 VS Code 后执行下列步骤：
1) 在 VS Code 活动栏（左侧菜单）中点击 **PlatformIO 徽标**。
2) 在 **Quick Access** 菜单中，在 **Miscellaneous** 下，选择 **New Terminal**。终端视口将会在一个被标记为 **PlatformIO CLI** 新终端中被加载。
{{< img "pio-new_terminal-alexa_intro.en.png" "PlatformIO CLI terminal in VS Code" "1 - 打开 PIO 菜单，2 - 打开新的 PIO 终端，3 - 验证你处于 'PlatformIO CLI' 终端会话中">}}

## 下载和安装 AWS CLI
AWS 命令行接口 (CLI) 是用于管理 AWS 服务的统一工具。只需下载和配置一个工具，您就可以使用命令行控制多个 AWS 服务并利用脚本来自动执行这些服务。要配置 AWS CLI，您首先需要有一个 AWS 账户。请先 [登录到控制台](https://console.aws.amazon.com/console/home) 或 [创建账户](https://portal.aws.amazon.com/billing/signup#/start)，然后再继续。

{{%expand "Ubuntu Linux v18.0+ (64-bit)" %}}
下载和安装最新版本的 AWS CLI version 2 for 64-bit Linux:
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```
{{% /expand%}}

{{%expand "MacOS 10.14+" %}}
1) 下载和安装最新版本的 AWS CLI version 2 installer for macOS [这里](https://awscli.amazonaws.com/AWSCLIV2.pkg)。
2) 双击下载的安装文件并按照显示的安装步骤进行安装。推荐你为所有用户安装 AWS CLI，否则，请参考 [此步骤](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-mac.html#cliv2-mac-install-gui) 创建符号链接并添加文件到你可执行路径中。
{{% /expand%}}

{{%expand "Windows 10 (64-bit)" %}}
1) 下载最新版本的 AWS CLI version 2 installer for Windows [这里](https://awscli.amazonaws.com/AWSCLIV2.msi)。
2) 双击下载的安装文件，按照显示的步骤安装。
{{% /expand%}}

## 检索 IAM 用户访问凭证
[IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) 是一个帮助你安全的控制 AWS 资源访问的 web 服务。推荐你 [创建一个管理员用户](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html)  而不要使用根账户。

参照下面的 [官方文档](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)，以检索您的 IAM 用户访问凭证。

## 配置 AWS CLI
在 AWS CLI 已经安装完毕并且 IAM 用户凭证已经获取之后，开始进行 AWS CLI的配置。需要配置的其中一个设定为 AWS [区域(region)](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)。需要留意是，在本教程中，区域的设置应保持不变，我们使用 **us-west-2**。使用不同的区域或者中途改变区域可能会导致接下来的步骤中出现问题，比如 [区域服务的可用性](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)。

在您的终端中输入下面的命令来进行 AWS CLI 的配置：
```bash
aws configure
```

这个命令将会提示你输入四个参数。类似如下的示例，输入您之前从 IAM 用户内获取到的 access key Id 和 secret access key：
```bash
AWS Access Key ID [None]: EXAMPLEKEYIDEXAMPLE
AWS Secret Access Key [None]: EXAMPLEtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## 测试 AWS CLI
上述配置完成之后，测试 AWS CLI 以确保其正常使用。首先，验证 AWS CLI 是否安装，然后验证 AWS CLI 的配置。

验证 CLI 正确安装，可以使用 version 选项。如果安装正常，则会输出 AWS CLI 版本（如果遇到错误，参考[排错指南](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)）：
```bash
aws --version
```

接下来，验证 AWS CLI 使用了你提供的 IAM 凭证和 US West (Oregon) 区域。你将会使用一个检查你的 AWS IoT [MQTT broker](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) 终端节点的命令，这个命令应该返回类似 `xxxxxxxx-ats.iot.us-west-2.amazonaws.com` 的地址。（如果遇到错误，参考 [排错指南](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)）。
```bash
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```


所有的安装和配置已经完成，我们进入到下一章节，[**设备预置**](device-provisioning.html)。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}
