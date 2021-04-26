+++
title = "Flashing Espressif Certificates"
weight = 20
pre = "<b>b. </b>"
+++

## Chapter introduction
By the end of this chapter, you will have the companion phone applications installed on your mobile device, cloned the necessary code repository, received certificates for connecting to AWS IoT using Espressif Alexa's AWS account, and flashed the certificates to separate flash partitions of the reference hardware.

## Companion Mobile Applications
In order to complete the authentication with Alexa, you will need Espressif's companion app to provision the device to Wi-Fi and provision the reference hardware with your Alexa account.

Download the ESP Alexa Phone App:
[iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

It is recommended (_optional_ for this tutorial) to have the Amazon Alexa app available for [iOS](https://apps.apple.com/us/app/amazon-alexa/id944011620) and [Android](https://play.google.com/store/apps/details?id=com.amazon.dee.app) - this is the same app used to provision most Alexa-enabled devices.

## Accessing the code
All the code for this tutorial is located in the `Alexa_for_IoT-Intro` folder from the repo you cloned in the [**Blinky Hello World**](/en/blinky-hello-world.html) tutorial. If you already have the repository locally, you can skip this section.

To clone the repo again from the [PlatformIO CLI terminal window](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window):
```
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```

## Opening the Project Environment
For this tutorial, you will use the Alexa_For_IoT-Intro project. In your new VS Code window, 
1. Click the **PlatformIO logo** in the VS Code activity bar (left most menu)
2. Select **Open** from the left PlatformIO menu
3. Click **Open Project**
4. Navigate to the `Core2-for-AWS-IoT-EduKit/Alexa_For_IoT-Intro` folder, and click **open**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Open PIO menu, 2 - Open PIO home, 3 - Open the project folder" >}}

Next, you'll need to open a new PlatformIO CLI terminal window in VS Code by:
1) Click the **PlatformIO logo** on the VS Code activity bar (left most menu).
2) From the **Quick Access** menu, under **Miscellaneous**, select **New Terminal**.

{{< img "pio-new_terminal-smart_thermostat.en.png" "PlatformIO CLI terminal in VS Code" "1 - Open PIO menu, 2 - Open new PIO Terminal, 3 - Verify you're in the 'PlatformIO CLI' terminal session, 4 - Paste the commands into terminal, 5 - If you encounter an error autodetecting the port, open the Platform.ini file and follow instructions to manually add the serial port.">}}

## Set up AWS IoT Certificates 
You need to create the AWS IoT credentials to communicate with AWS IoT core. For this workshop and device, Espressif has provided the AWS IoT credentials that can be used on their AWS account with the M5Stack Core2 for AWS IoT EduKit reference hardware. To obtain the credentials for your device to connect to their service, fill out the form [here](https://espressif.github.io/esp-va-sdk/).

After you receive the e-mail containing the credentials zip file, you will need to save the file and unzip the contents. After extraction, you will have a folder called **espcredentials**. With the device plugged in, you can upload these certificates to your device by entering the following commands in your PlatformIO CLI terminal window:

{{%expand "Ubuntu or macOS" %}}
1. Copy the contents of espcredentials to the `Core2-for-AWS-IoT-EduKit/Alexa_for_IoT-Intro/esp_alexa_credentials` folder *except* the **mfg_config.csv** file. Replace **<<PATH_TO>>** with the location of the **espcredentials** folder. The trailing **/** of the source tells [rsync](https://download.samba.org/pub/rsync/rsync.1) to copy the contents to the `./esp_alexa_credentials/` destination. Excluding it will copy the folder and cause file not found errors in the next step:
   ```bash
   rsync -avr --exclude='mfg_config.csv' <<PATH_TO>>/espcredentials/ ./esp_alexa_credentials/
   ```
   
2. Use the PIO command below to run a script which will 1) create a binary file with the credential files called `mfg.bin` in the esp_alexa_credentials folder, and then upload the binary to a partition of the on-board [non-volitile flash storage](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/storage/nvs_flash.html) (NVS) and later be accessed by the application as a key:value pair:
   ```bash
   pio run --environment core2foraws --target flash_certificates
   ```
{{% /expand%}}
{{%expand "Windows" %}}
1. Copy the contents of espcredentials to the `Core2-for-AWS-IoT-EduKit/Alexa_for_IoT-Intro/esp_alexa_credentials` folder *except* the **mfg_config.csv** file. Replace **<<PATH_TO>>** with the location of the **espcredentials** folder. The trailing **/** of the source tells [rsync](https://download.samba.org/pub/rsync/rsync.1) to copy the contents to the `./esp_alexa_credentials/` destination. Excluding it will copy the folder and cause file not found errors in the next step:
   ```PowerShell
   robocopy "<<PATH_TO>>\espcredentials\" ".\esp_alexa_credentials\" /xf mfg_config.csv
   ```
   
2. Use the PIO command below to run a script which will 1) create a binary file with the credential files called `mfg.bin` in the esp_alexa_credentials folder, and then upload the binary to a partition of the on-board [non-volitile flash storage](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/storage/nvs_flash.html) (NVS) and later be accessed by the application as a key:value pair:
   ```PowerShell
   pio run --environment core2foraws --target flash_certificates
   ```
{{% /expand%}}

With everthing set up and ready, let's move on to [**Building and Testing AFI**](/en/intro-to-alexa-for-iot/building-and-testing-afi.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}