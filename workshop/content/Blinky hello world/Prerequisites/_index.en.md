+++
title = "Prerequisites"
weight = 10
pre = "<b>a. </b>"
+++

In this section, you download and install the [AWS CLI](https://aws.amazon.com/cli/) for your host machine's operating system, retrieve AWS [Identity and Access Management](https://aws.amazon.com/iam/) (IAM) user access credentials to manage services with the AWS CLI, configure the AWS CLI, and finally test that the AWS CLI is working properly. 

{{% notice info%}}
If you already have the AWS CLI (version 1 or version 2) installed and configured on your machine, skip to [Test the AWS CLI](#testing-the-aws-cli).
{{% /notice %}}

## Open the PlatformIO CLI terminal window
In the [Getting Started](../getting-started.html) tutorial, you installed and used Platform IO (PIO) and the PIO terminal window. The PIO terminal window pre-loads additional applications and libraries that your standard terminal/command prompt might not have. It is important to continue to use the PIO terminal window for all subsequent steps. 

If you closed Visual Studio Code (VS Code) or don't have the terminal viewport with the PlatformIO CLI displayed, complete the following the steps:
1. Open **VS Code**, if necessary.
1. Choose the **PlatformIO logo** on the VS Code activity bar.
1. From the **Quick Access** menu, expand **Miscellaneous**, and select **New Terminal**. The terminal viewport should load with a new terminal labeled *PlatformIO CLI*.
{{< img "pio-new_terminal-alexa_intro.en.png" "PlatformIO CLI terminal in VS Code" "1 - Open PIO menu, 2 - Open a new PIO Terminal, 3 - Verify that you have a 'PlatformIO CLI' terminal session">}}

## Download and Install the AWS CLI
The AWS Command Line Interface (CLI) is a unified tool to manage your AWS services. You can control multiple AWS services from the command line and automate them through scripts with just one tool . To configure the AWS CLI, you first need to have an AWS account. 

* If you do not have a personal AWS account, please [sign in to AWS console](https://console.aws.amazon.com/console/home) or [create an AWS account](https://portal.aws.amazon.com/billing/signup#/start) before proceeding.
* Once complete, expand the appropriate operating system from the following list and complete the instructions to install the AWS CLI:

{{%expand "Ubuntu Linux v18.0+ (64-bit)" %}}
Issue the following commands to download and install the latest AWS CLI version 2 for 64-bit Linux:
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```
{{% /expand%}}

{{%expand "MacOS 10.14+" %}}
1. Download the latest [ AWS CLI version 2 installer for macOS](https://awscli.amazonaws.com/AWSCLIV2.pkg).
1. Launch the downloaded installer and follow the on-screen instructions to install the AWS CLI on the host machine. We recommend you install the AWS CLI for all users, but if you choose not to, follow [these instructions](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-mac.html#cliv2-mac-install-gui) to create a symlink and add the file to your path.
{{% /expand%}}

{{%expand "Windows 10 (64-bit)" %}}
1. Download the latest [AWS CLI version 2 installer for Windows](https://awscli.amazonaws.com/AWSCLIV2.msi).
1. Launch the downloaded installer and follow the on-screen instructions to install the AWS CLI on your host machine.
{{% /expand%}}

For additional information and instructions, see [Installing or updating the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

## Retrieve the IAM user access credentials
[AWS Identity and Access Management](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) (IAM) is a web service that helps you securely control access to AWS resources. It is considered a best practice to [create an administrative user](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) instead of using your root user account.

To retrieve your IAM user's access credentials, follow the instructions outlined in the [Quick configuration with aws configure](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) documentation.

## Configure the AWS CLI
With the AWS CLI installed and the IAM user access credentials in hand, it's time to configure the AWS CLI. 

One of the settings you'll configure is the [AWS Region](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html). It is important to keep in mind that the Region you're currently using stays consistent—for purposes of this tutorial, we use `us-west-2` as a standard Region. Using a different Region or unknowingly changing Regions can cause other challenges in subsequent steps; such as, [regional service availability](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/).

Issue the following command in the terminal viewport to configure the AWS CLI on your host machine:
```bash
aws configure
```

The CLI will prompt you to input four parameters. Complete the information similar to the following example. 
**Note:** the AWS Access Key ID and AWS Secret Access Key will need to match the information that you retrieved for your IAM user. 

- AWS Access Key ID [None]: `EXAMPLEKEYIDEXAMPLE`
- AWS Secret Access Key [None]: `EXAMPLEtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`
- Default region name [None]: `us-west-2`
- Default output format [None]: `json`

## Test the AWS CLI
With the AWS CLI configured, it is time to test that everything is configured correctly. Let's begin with a simple test. Execute the following command to confirm the AWS CLI version that's installed:

```bash
aws --version
```
{{% notice note %}}
 If you receive errors, see [Troubleshooting AWS CLI errors](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html).
{{% /notice %}}

Next, verify that the AWS CLI is configured with your IAM credentials and is using the us-west-2 (US West (Oregon)) Region. Execute the following command to check your [MQTT broker](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) endpoint for AWS IoT. 
```bash
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```
It should return an address with the pattern `xxxxxxxx-ats.iot.us-west-2.amazonaws.com`. Take a moment to copy and paste the endpoint to a secure location. If you receive an error, see the troubleshooting guide noted above.

Now that everything is installed and configured, continue to the next chapter, [**Device Provisioning**](device-provisioning.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}