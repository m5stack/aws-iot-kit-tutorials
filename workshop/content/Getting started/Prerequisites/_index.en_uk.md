+++
title = "Prerequisites"
weight = 11
pre = "<b>a. </b>"
+++

In this chapter, we download and install the software that you will need to view, edit, compile a binary firmware, and transfer the firmware to the device for it to run. Additionally, you'll be downloading an app on your phone to control the device.

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
{{%expand "Windows" %}}
1) Disconnect the USB-C cable connecting your Core2 for AWS IoT EduKit device to your computer if it's already connected.
2) Download and extract the [CP210x Universal Windows driver](https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip).
3) Open **CP210xVCPInstaller_x64.exe** and run through the installer to install the drivers.
4) Reboot your computer and reconnect your Core2 for AWS IoT EduKit device to your computer via the provided USB-C cable.
{{% notice warning %}}
The tutorials were only tested on Windows 10 64-bit. We do not support any other configuration of the Windows operating system.
{{% /notice %}}
{{% /expand%}}

## Visual Studio Code installation
Visual Studio Code is an open source integrated development environment (IDE) which allows you to view, edit, manage code and more. Download the latest [Visual Studio Code](https://code.visualstudio.com/) for your operating system. For troubleshooting issues with Visual Studio Code installation or usage, please refer to [their documentation](https://code.visualstudio.com/docs/setup/setup-overview).

## Installing PlatformIO
[PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) provides a professional embedded development platform which simplifies embedded software development. The Visual Studio Code extension provides the functionality of the Platform IO command line interface (CLI) in a graphical interface. You can download the extension and read more about PlatformIO [here](https://platformio.org/install/ide?install=vscode).

{{%expand "Windows" %}}
{{% notice info %}}
If you already have Espressif IoT Development Framework (ESP-IDF) installed on your computer, you might see an error about missing files or unable to install dependencies such as the **esp-windows-curses**. This is due to conflicts. To resolve the issue at this time, please reference [this post](https://community.platformio.org/t/cant-create-esp-idf-project-correctly-in-platformio/16370/17) on the PlatformIO Community forum.
{{% /notice %}}
{{% /expand%}}

## Downloading and installing the phone apps.
The ESP RainMaker Phone Apps are available for iOS and Android phones to provide Wi-Fi network configuration, user-creation, user-device association and device control. The apps can be found here:
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker), [Direct APK](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

If you do not posess a compatible Android or iOS device, you can use the [Rainmaker CLI](https://rainmaker.espressif.com/docs/cli-setup.html) and substitute instructions.

## Downloading the code
You can either download the code you'll be using throughout this program by cloning the repository hosted on [GitHub](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit), or download and extract the [zip file](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/archive/master.zip). To clone:
```bash
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```
{{%expand "Windows" %}}
{{% notice info %}}
Windows has file path length limitations that can cause errors depending on where the project is placed. It is recommended that you clone this repo or extract the zip with a shorter initial path length if this error should occur (e.g. **C:\\**).
{{% /notice %}}
{{% /expand%}}

## Connecting and powering on the device
Lastly, let's make sure the device is plugged in to your computer and powered on. The device should automatically turn on once plugged in and connected to a power source, but if you need to turn it on, press the power button.
![How to turn M5Stack Core2 for AWS on or off](prerequisites/core2foraws_power_on_off.jpg?width=500px&classes=shadow)

With the device ready, let's go to the next chapter — [**Running the ESP RainMaker Agent**](/en_uk/getting-started/run-rainmaker.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}