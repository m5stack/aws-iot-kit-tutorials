+++
title = "Prerequisites"
weight = 10
pre = "<b>a. </b>"
+++

In this chapter, you'll install required software for this and all subsequent tutorials. Starting with the SiLabs CP210x drivers to communicate with the Core 2 for AWS IoT EduKit over USB, then the ESP-IDF toolchain for the on-board ESP32-D0WD microcontroller unit (MCU). You will also install Miniconda to manage your Python versions and your dependencies in isolated virtual environments to avoid conflicts. Additionally, we will download, install, and configure the AWS command line interface (CLI). This tutorial assumes that you have an [AWS account](https://signin.aws.amazon.com/signin).

## Silicon Labs USB to UART bridge driver installation
Download and install the SiLabs CP210x drivers to enable your computer to communicate with the Core2 for AWS IoT EduKit device. The on-board CP2104 is an USB-to-UART bridge to facilitate host communication with the ESP32-D0WD microcontroller:
{{%expand "macOS 10.9+" %}}
Since OS X Mavericks, Apple has included the necessary USB to serial drivers and no other steps are necessary. To verify that the drivers are installed, loaded, and the device is read for programming, connect the device via the provided USB-C cable and run:
```bash
ls -l /dev/cu.S*
```
If your host machine recognizes and is able to communicate with the device, you will see a return value of `/dev/cu.SLAB_USBtoUART` printed. If the results of the command above returns empty, check the physical connection first. If it doesn't resolve the issue, follow the instructions for macOS <= 10.8.
{{% notice info %}}
On macOS 10.13 and later, the installation of the SiLabs system extension may be blocked. To unblock, open your Mac's **System Preferences** <i class="fas fa-arrow-right"></i> **Security & Privacy** pane, unlock changes by clicking the <i class="fas fa-lock"></i>, **allow** the developer, and then relock by clicking the <i class="fas fa-lock-open"></i>. For more information, see [Apple Technical Note TN2459](https://developer.apple.com/library/archive/technotes/tn2459/_index.html).
{{% /notice %}}
{{% /expand%}}
{{%expand "macOS <= 10.8" %}}
1) Disconnect the USB-C cable connecting your Core2 for AWS IoT EduKit device to your computer if it's already connected.
2) Download and extract the [CP210x VCP macOS driver](https://www.silabs.com/documents/public/software/Mac_OSX_VCP_Driver.zip).
3) Expand the **SiLabsUSBDriverDisk.dmg** file from within the extracted folder.
4) Open the **Install CP210x VCP Driver** application and run through the installer.
5) Reboot your computer and reconnect your Core2 for AWS IoT EduKit device to your computer via the provided USB-C cable.
{{% /expand%}}
{{%expand "Linux" %}}
Linux kernel version 3.x.x and 4.x.x already include the drivers as part of the distribution. To verify they are installed and loaded, run the command in your terminal:
```bash
modinfo usbserial
modinfo cp210x
```
If they are *not* loaded, run the command:
```bash
sudo modprobe usbserial
sudo modprobe cp210x
```
{{% /expand%}}
{{%expand "Windows (64-bit)" %}}
1) Disconnect the USB-C cable connecting your Core2 for AWS IoT EduKit device to your computer if it's already connected.
2) Download and extract the [CP210x Universal Windows driver](https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip).
3) Open **CP210xVCPInstaller_x64.exe** and run through the installer to install the drivers.
4) Reboot your computer and reconnect your Core2 for AWS IoT EduKit device to your computer via the provided USB-C cable.
{{% notice warning %}}
The tutorials were only tested on the 64-bit version of Windows 10. We do not support any other configuration of the Windows operating system.
{{% /notice %}}
{{% /expand%}}

## Installing ESP-IDF v4.2
The **M5Stack Core2 for AWS** reference hardware you're using is equipped with an Espressif ESP32-D0WD microcontroller unit (MCU). The ESP-IDF (Espressif IoT Development Framework) allows you to configure, build, and flash firmware onto an ESP32 board. In the Getting Started tutorial, you used PlatformIO to simplify compilation and flashing of the firmware on the device—which provides a GUI to the behind-the-scene calls to ESP-IDF version 4.1 and other tools. For the subsequent steps, you will use ESP-IDF *version 4.2* due to enhancements to the mbedTLS library and improved integration of the secure element.

{{%expand "macOS" %}}
Copy and paste the command block below to:
1) Install Homebrew if you do not have it yet.
2) Install cmake, ninja, and the DFU utility.
3) Clone the ESP-IDF into the `$HOME/esp/` directory (the full ESP-IDF path is **$HOME/esp/esp-idf**).
4) Install the ESP-IDF dependencies.
5) Check your installation and export the ESP-IDF environmental variables to your **$PATH**.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install cmake ninja dfu-util
mkdir $HOME/esp
cd $HOME/esp
git clone -b release/v4.2 --recursive https://github.com/espressif/esp-idf.git
cd $HOME/esp/esp-idf
. $HOME/esp/esp-idf/install.sh
. $HOME/esp/esp-idf/export.sh
```
{{% /expand%}}
{{%expand "Linux" %}}
1) Install all your dependencies.
   - CentOS 7:
   ```bash
   sudo yum install git wget flex bison gperf cmake ninja-build ccache dfu-util
   ```
   - Ubuntu and Debian:
   ```bash
    sudo apt-get install git wget flex bison gperf python-setuptools cmake ninja-build ccache libffi-dev libssl-dev dfu-util
    ```
    - Arch:
    ```bash
    sudo pacman -S --needed gcc git make flex bison gperf cmake ninja ccache dfu-util
    ```
2) Clone the ESP-IDF into the `$HOME/esp/` directory (the full ESP-IDF path is $HOME/esp/esp-idf).
3) Install the ESP-IDF dependencies.
4) Check your installation and export the ESP-IDF environmental variables to your **$PATH**.

```bash
mkdir $HOME/esp

cd $HOME/esp

git clone -b release/v4.2 --recursive https://github.com/espressif/esp-idf.git

cd $HOME/esp/esp-idf

. $HOME/esp/esp-idf/install.sh

. $HOME/esp/esp-idf/export.sh
```
{{% /expand%}}
{{%expand "Windows (64-bit)" %}}
1) Download and run the [ESP-IDF Tools Installer](https://dl.espressif.com/dl/esp-idf-tools-setup-2.3.exe) to install **ESP-IDF v4.2** and all other dependencies.
2) In the **Python choice** stage of the installer, select **Install Python 3.7**. If Python 3.7 is already installed, you might not have the install option—select your installed Python 3.7 (do *not* use the Python installed with Anaconda/miniconda if that is an available option).
3) In the **Download ESP-IDF** stage of the installer to download, select **release/v4.2 (release branch)**.
4) Reboot your machine if you haven't done so yet after the installer ran.
5) `cd %userprofile%\Desktop\esp-idf`
6) Run the ESP-IDF installation script to ensure everything is set correctly: `install.bat`
7) Run the export script add ESP-IDF tools to your path: `export.bat`
{{% /expand%}}

## Installing and configuring the AWS CLI version 2
### AWS CLI Installation
The AWS Command Line Interface (CLI) is a unified tool to manage your AWS services. With just one tool to download and configure, you can control multiple AWS services from the command line and automate them through scripts. To be able to configure the AWS CLI, you'll first need to have an AWS account. Please [signin](https://signin.aws.amazon.com/signin) or [create an account](https://portal.aws.amazon.com/billing/signup#/start) first before proceeding. After you've signed in follow the official [AWS CLI installation instructions for your OS](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

### AWS CLI Configuration
Once you have that installed, it is time to set it for your account and region. It's important to keep in mind that the region you're currently using stays consistent—for purposes of this tutorial, we are standardizing on **us-west-2**. Changing regions can cause other challenges in subsequent steps. To view regional service availability, reference the [AWS Regional Services List](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) for a list of services that are available.

Use the command:
```bash
aws configure
```
For steps on obtaining the credentials for the AWS CLI, view the [quick configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) documentation. To get the AWS access key Id and AWS secret access key, you will need to give necessary [permissions to your IAM user](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config). Your inputs should look like the following:
```
AWS Access Key ID [None]: EXAMPLEKEYIDEXAMPLE
AWS Secret Access Key [None]: EXAMPLEtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## Miniconda setup and installation
Miniconda is a bootstrap version of the full-fledge Anaconda—which simplifies package and Python virtual environment management. Many installation errors are due to incompatible Python versions (e.g. Python 2.x), the python PATH variable being set to the wrong Python version, or dependency conflicts. It is *strongly* encouraged to always use Conda to handle your Python development environment.

Go to https://docs.conda.io/en/latest/miniconda.html and download the corresponding installer for your OS. Follow the official Miniconda installation instructions and then return to this page.

{{%expand "macOS and Linux" %}}
1) After miniconda is installed, quit and then reopen your terminal application so that the source files are added and your PATH includes the environment variables you'll need.
2) Creating a new conda environment to isolate your work and avoid dependency conflicts. After running the command below, enter **y** to install the necessary dependencies for the environment:
   ```bash
   conda create -n edukit python=3.7
   ```
3) After successfully creating the new environment, you'll switch out of the *base* conda environment and into the newly created environment. You'll know you're activated because you'll see **(edukit)** at the beginning of the current prompt:
   ```bash
   conda activate edukit
   ```
4) You'll run the ESP-IDF toolchain export script to add the ESP tools to your path:
    ```bash
    . $HOME/esp/esp-idf/export.sh
    ```
{{% /expand%}}
{{%expand "Windows (64-bit)" %}}
1) After the miniconda installer has finished, reboot your machine.
2) Open Anaconda Prompt by going to **start menu** → **Anaconda prompt**.
3) Creating a new conda environment to isolate your work and avoid dependency conflicts. After running the command below, enter **y** to install the necessary dependencies for the environment:
   ```bash
   conda create -n edukit python=3.7
   ```
4) After successfully creating the new environment, you'll switch out of the *base* conda environment and into the newly created environment. You'll know you're activated because you'll see **(edukit)** at the beginning of the current prompt:
   ```bash
   conda activate edukit
   ```
5) Run the export script add ESP-IDF tools to your path: 
   ```bash
   %userprofile%\Desktop\esp-idf\export.bat
   ```
{{% /expand%}}

{{% notice note %}}
If you close your shell or open a new shell, you'll need to re-enter `conda activate edukit` to reactivate the virtual environment and run ESP-IDF's `export.sh` (macOS/Linux) or `export.bat` (Windows) to re-add the ESP-IDF tools to your path each time.
{{% /notice %}}

## Cloning the code repository
If you have misplaced the code repository from the [Getting Started Prerequisites](/en_uk/getting-started/prerequisites.html) instructions, you can clone the repository hosted on [GitHub](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit), or download and extract the [zip file](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/archive/master.zip). To clone:
```bash
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```
{{%expand "Windows" %}}
{{% notice info %}}
Windows has file path length limitations that can cause errors depending on where the project is placed. It is recommended that you clone this repo or extract the zip with a shorter initial path length if this error should occur (e.g. **C:\\**).
{{% /notice %}}
{{% /expand%}}

With everything installed and configured, let's move to the next chapter, [**Device Provisioning**](device-provisioning.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}