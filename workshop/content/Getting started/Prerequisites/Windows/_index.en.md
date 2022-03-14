+++
linkTitle = "Windows"
title = "Windows 10 Set Up Instructions"
pre = "› "
+++

This section provides information to configure your Windows computer (host machine) to download, view, and edit code from the AWS IoT EduKit GitHub repository. When you complete these steps, your computer will be ready to compile and upload code to the hardware's flash memory. Finally, these steps are required to install the ESP RainMaker agent.

## Install Git and its dependencies
 Git is a widely adopted distributed version control system, and is commonly used for source code management and collaboration. Git also allows users to track file changes, and distribute code between a local machine and remote server (for more information, see [About Git](https://git-scm.com/about)).

OpenSSL provides a toolkit for secure communication to the Git repository (for more information, see [OpenSSL](https://www.openssl.org/))
 
Complete the following steps to install OpenSSL, Git, and Git's dependencies:
1. Download and install [OpenSSL](https://slproweb.com/products/Win32OpenSSL.html).
1. Download and install [Git for Windows](https://git-scm.com/download/win).
   * When you install Git for Windows, use the default options and be sure to choose **Use the OpenSSL library** in the *Choosing HTTPS transport backend installer* page.

![Git for Windows installation wizard. Choose Use the OpenSSL library](windows/git-for-windows-openssl2.png?width=450px&classes=shadow)

## Set up Silicon Labs USB-to-UART bridge
The AWS IoT EduKit communicates with the host machine through a Silicon Labs CP210x USB-to-UART bridge. The on-board CP2104 bridge that facilitates host communication with the ESP32-D0WD microcontroller. The microcontroller communicates bi-directionally over [UART](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/uart.html)0, which the CP210x bridge translates through a virtual communication port on the host machine. 

To be able to mount the virtual serial port and communicate across it, you must download and install the supporting driver. Complete the following steps to install the driver for the CP2104 bridge:
1) Ensure the AWS IoT EduKit device is not connected to host machine.
2) Download the [Windows Silicon Labs CP210x](https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip) driver. (For more information, see the [Silicon Labs, Downloads](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) page.)
3) Extract the downloaded file.
   {{% notice note %}}
   You must extract the contents of the folder because the driver will not install if the executable runs within the archive. Also, it is not important where you save the extracted folder. 
   {{% /notice %}} 
4) **Open the folder**, right-click the **silabser.inf** file, and choose **Install**.
5) Restart your host machine to make sure the driver is applied.

## Install Visual Studio Code
Visual Studio Code (VS Code) is an open source integrated development environment (IDE) that allows you to view, edit, and manage code. 

* Download and install the latest [VS Code](https://code.visualstudio.com/) software for your operating system. To troubleshoot issues with Visual Studio Code's installation or use, refer to [Setting up Visual Studio Code](https://code.visualstudio.com/docs/setup/setup-overview) in their documentation.

## Install PlatformIO
[PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) (PIO) provides a professional embedded development platform that simplifies embedded software development. This VS Code extension combines the functionality of the Platform IO command line interface (CLI) with a graphical user interface (GUI). 

Complete the steps outlined in [PlatformIO's installation instructions](https://platformio.org/install/ide?install=vscodeDownload) to download and install the VS Code extension. 

Restart VS Code after the PIO installation completes.

## Clone the code repository
All of the projects exist in a [GitHub repository](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/about-repositories). Through this repository you can clone the instructions, and view the revision history of each file in the repository.

Complete the following steps to clone the code for the tutorials:
1. Choose the **PlatformIO logo** on the VS Code activity bar to open PlatformIO's Quick Access menu. 
1. Expand **Miscellaneous** and choose **Clone Git Project**.
1. Copy the following URL and paste it into the target field at the top of the page.
   ```
   bash https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
   ```
1. Select the location that you want to save the project.
{{< img "pio-clone_git_project.en.png" "PlatformIO Clone Git Project" "1 - Open PIO menu, 2 - Choose Clone Git Project, and 3 - Paste the repository URL" >}}

## Download and install the phone apps
The ESP RainMaker Phone Application is available for iOS and Android phones to provide Wi-Fi network configuration, user-creation, user-device association, and device control. 

Install the ESP Rainmaker application through:
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker), [Direct APK](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

If you don't have a compatible Android or iOS device, follow the [ESP RainMaker CLI Setup](https://rainmaker.espressif.com/docs/cli-setup.html) instructions.

## Identify the device communication port
If you haven't already, unbox the AWS IoT EduKit and connect it to your host computer's USB 2.0 port using the supplied USB-A to USB-C cable. (You do not need to use the hex key that is included at this time. This key is used to install additional modules that are sold separately.) 

The device should automatically turn on when you plug it in. If it doesn't, press the power button.
![How to turn M5Stack Core2 for AWS on or off](windows/core2foraws_power_on_off.jpg?width=500px&classes=shadow)

Now that the device is ready and the prerequisite software is installed, let's identify the virtual port your device is using so that you can correctly route read and write operations.

1) From PlatformIO's **Quick Access** menu, expand **PIO Home**, and select **Devices**.
2) Choose the icon next to the port with the description **Silicon Labs CP210x USB to UART Bridge** (this is usually `COM3`). **Write down the device port number.** (**Note:** When you disconnect and reconnect the AWS IoT EduKit to your host machine, the port number can change. Be sure to confirm this port each  time.)

{{% notice note %}}
If your AWS IoT EduKit does not appear in the device list, confirm that it's powered on and you are using the supplied USB-A to USB-C cable. Some USB-C hubs have compatibility issues with establishing a serial port.
{{% /notice %}}

## Next
Now that your host machine can communicate with the AWS IoT EduKit reference hardware, continue to the next chapter — [**Run the ESP RainMaker Agent**](/en/getting-started/run-rainmaker.html).

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}