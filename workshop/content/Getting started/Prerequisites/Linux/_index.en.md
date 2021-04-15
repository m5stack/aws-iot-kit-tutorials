+++
pre = "› "
linkTitle = "Linux"
title = "Ubuntu Linux v18.0+ Setup Instructions"
+++

Below are the instructions to set up your linux computer (host machine) to be able to download the code from the GitHub repository, view and edit the code, compile the code to be usable by the hardware, and upload the code to the hardware's flash memory. These installation steps are sufficient for the **Getting Started** tutorial, which uses Espressif's AWS account and services for the RainMaker platform.

## Install dependencies
In order to download the code, [compile the code](https://en.wikipedia.org/wiki/Object_code), and run scripts to download future dependencies using the [Python programming language's package installer](https://pip.pypa.io/en/stable/), you'll need to install some dependency libraries or applications first. To download the code from the remote code repository on GitHub, you'll need to install [git](https://git-scm.com/). To compile the code, Espressif uses the [cmake](https://cmake.org/) build system to parse the configuration of the device application and the [gcc compiler](https://gcc.gnu.org/onlinedocs/gcc/) to compile the firmware to object code that the Core2 for AWS IoT EduKit can understand and run. All the dependencies can be easily installed using apt via the following command in terminal:
```bash
sudo apt install build-essential python3-pip curl git cmake
```

## Silicon Labs USB to UART bridge setup
The Core2 for AWS IoT EduKit communicates with the host machine through a Silicon Labs CP210x USB to UART bridge. The on-board CP2104 is an USB-to-UART bridge to facilitate host communication with the ESP32-D0WD microcontroller. The microcontroller communicates bi-directionally over [UART](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/uart.html)0, which the CP210x translates through a virtual communications port on the host machine it establishes over USB-C.

Linux kernel version 3.x.x and 4.x.x already include the drivers needed for the CP210x as part of the distribution. However, by default, serial communications is restricted to users that are part of the dialout group. To add your user to the dialout group, use terminal and enter the following:
```bash
sudo usermod -a -G dialout $USER
```

You need to restart your host machine *now* to have the permissions applied.

## Visual Studio Code installation
Visual Studio Code (VS Code) is an open source integrated development environment (IDE) which allows you to view, edit, manage code and more. Download the latest [Visual Studio Code](https://code.visualstudio.com/) for your operating system. For troubleshooting issues with Visual Studio Code installation or usage, please refer to [their documentation](https://code.visualstudio.com/docs/setup/setup-overview).

## Installing PlatformIO
[PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) (PIO) provides a professional embedded development platform which simplifies embedded software development. The Visual Studio Code extension provides the functionality of the Platform IO command line interface (CLI) in a graphical interface. You can download the extension and read more about PlatformIO [here](https://platformio.org/install/ide?install=vscode).

You need to restart VS Code after PlatformIO extension installation finishes.

## Cloning the code repository
All of the projects and files exist in a [GitHub repository](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/about-repositories), where you can also view the revision history of each file in the repository (repo). To clone the code you'll need for the tutorials, you'll be using the PIO interface by:
1) Clicking the PlatformIO logo on the VS Code activity bar (left most menu).
2) From PlatformIO's **Quick Access** menu, under **Miscellaneous**, select **Clone Git Project**.
3) Paste `https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git` into the text field and then select the location you want to save the project in.
{{< img "pio-clone_git_project.en.png" "PlatformIO Clone Git Project" "1 - Open PIO menu, 2 - Clone git project, 3 - Paste repository URL" >}}

## Downloading and installing the phone apps
The ESP RainMaker Phone Apps are available for iOS and Android phones to provide Wi-Fi network configuration, user-creation, user-device association and device control. The apps can be found here:
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker), [Direct APK](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

If you do not posess a compatible Android or iOS device, you can use the [RainMaker CLI](https://rainmaker.espressif.com/docs/cli-setup.html) and substitute the provided instructions.

## Identifying the device communication port
If you haven't already, it's time to unbox the Core2 for AWS IoT EduKit reference hardware and connect it to your host computer's USB 2.0 port using the supplied USB-A to USB-C cable to facilitate communication between them. Additionally, included in the package is a hex key that can be used for the installation of additional modules (sold separately). The device should automatically turn on once you plug it in, but if you need to turn it on, press the power button.
![How to turn M5Stack Core2 for AWS on or off](linux/core2foraws_power_on_off.jpg?width=500px&classes=shadow)

With the device ready and the software you'll need for this tutorial installed, let's identify the port your device is virtually mounted to so that you can perform read & write operations to that specific port.
1) From PlatformIO's **Quick Access** menu, under **PIO Home**, select **Devices**.
2) Click the icon next to the port with the description "CP2104 USB to UART Bridge Controller" (usually `/dev/ttyUSB0`) to copy the device port.

{{% notice note %}}
If your Core2 for AWS IoT EduKit does not show up in the list of devices, check that it's powered on and you are using the supplied USB-A to USB-C cable. Some USB-C hubs have compatibility issues with establishing a serial port.
{{% /notice %}}

## Next
With everything set up and your host machine ready and able to communicate with the Core2 for AWS IoT EduKit reference hardware, let's continue to the next chapter — [**Running the ESP RainMaker Agent**](/en/getting-started/run-rainmaker.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}