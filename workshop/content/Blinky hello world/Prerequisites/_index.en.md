+++
title = "Prerequisites"
weight = 10
pre = "<b>a. </b>"
+++

In this chapter, you'll download and install the [AWS CLI](https://aws.amazon.com/cli/) for your host machine's operating system, retrieve AWS [Identity and Access Management](https://aws.amazon.com/iam/) (IAM) user access credentials to manage services with the AWS CLI, configure the AWS CLI, and finally test that the AWS CLI is working properly. This tutorial assumes that you have an [AWS account](https://console.aws.amazon.com/console/home) and you have completed [setting up your environment](../getting-started/prerequisites.html). If you already have the AWS CLI (version 1 or version 2)installed and configured on your machine, skip to the [test section](#testing-the-aws-cli).

## Open the PlatformIO CLI Terminal Window
In the [Getting Started](../getting-started.html) tutorial, you installed and used PIO and used the PIO terminal window. It is important to continue to use the PIO terminal window for all subsequent steps. The PIO terminal window pre-loads additional applications and libraries that your standard terminal/command prompt might not have. 

If you've closed VS Code or don't have the terminal viewport with the PlatformIO CLI loaded in VS Code, follow the steps below after opening VS Code:
1) Click the **PlatformIO logo** on the VS Code activity bar (left most menu).
2) From the **Quick Access** menu, under **Miscellaneous**, select **New Terminal**. The terminal viewport should load with a new terminal labeled **PlatformIO CLI**.
{{< img "pio-new_terminal-alexa_intro.en.png" "PlatformIO CLI terminal in VS Code" "1 - Open PIO menu, 2 - Open new PIO Terminal, 3 - Verify you're in the 'PlatformIO CLI' terminal session">}}

## Downloading and Installing the AWS CLI
The AWS Command Line Interface (CLI) is a unified tool to manage your AWS services. With just one tool to download and configure, you can control multiple AWS services from the command line and automate them through scripts. To be able to configure the AWS CLI, you'll first need to have an AWS account. Please [sign in to AWS console](https://console.aws.amazon.com/console/home) or [create an AWS account](https://portal.aws.amazon.com/billing/signup#/start) first before proceeding.

{{%expand "Ubuntu Linux v18.0+ (64-bit)" %}}
Download and install the latest AWS CLI version 2 for 64-bit Linux:
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```
{{% /expand%}}

{{%expand "MacOS 10.14+" %}}
1) Download the latest AWS CLI version 2 installer for macOS [here](https://awscli.amazonaws.com/AWSCLIV2.pkg).
2) Double-click the downloaded installer file and follow the on-screen instructions to install on the host machine. It is recommended to install the AWS CLI for all users, but if you choose not to, follow [these instructions](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-mac.html#cliv2-mac-install-gui) to create a symlink and add the file to your path.
{{% /expand%}}

{{%expand "Windows 10 (64-bit)" %}}
1) Download the latest AWS CLI version 2 installer for Windows [here](https://awscli.amazonaws.com/AWSCLIV2.msi).
2) Double-click the downloaded installer file and follow the on-screen instructions to install on the host machine.
{{% /expand%}}

## Retrieve the IAM user access credentials
[IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) is a web service that helps you securely control access to AWS resources. It is recommended you [create an administrative user](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) first instead of using your root user account.

To retrieve your IAM user's access credentials, follow the [official docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config).

## Configuring the AWS CLI
With the AWS CLI installed and the IAM user access credentials in hand, it's time to configure the AWS CLI. One of the settings you'll set is the AWS [region](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html). It's important to keep in mind that the region you're currently using stays consistent—for purposes of this tutorial, we are standardizing on **us-west-2**. Using a different region or unknowingly changing regions can cause other challenges in subsequent steps, such as [regional service availability](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/).

To configure the AWS CLI on your host machine, enter the following command in the terminal viewport:
```bash
aws configure
```

The CLI will have prompts requesting inputs for four parameters. The fields should be filled out similar to below, with the corresponding access key Id and secret access key that was retrieved earlier for your IAM user:
```bash
AWS Access Key ID [None]: EXAMPLEKEYIDEXAMPLE
AWS Secret Access Key [None]: EXAMPLEtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## Testing the AWS CLI
With everything configured as described above, it is now time to test your AWS CLI to ensure it is working properly. First, you will verify the CLI is installed, and then validate the configuration.

To check the CLI is installed correctly, we will use the version option. A successful installation will output the AWS CLI version (if you receive errors, visit the [troubleshooting guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)):
```bash
aws --version
```

Next, you will verify the AWS CLI is configured with your IAM credentials and US West (Oregon) region. The command you will run will check your [MQTT broker](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) endpoint for AWS IoT. It should return an address with the pattern `xxxxxxxx-ats.iot.us-west-2.amazonaws.com`. If you receive errors, visit the [troubleshooting guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html).
```bash
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```


With everything installed and configured, let's move to the next chapter, [**Device Provisioning**](device-provisioning.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}