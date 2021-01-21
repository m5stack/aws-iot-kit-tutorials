+++
title = "Running the ESP RainMaker Agent"
weight = 12
pre = "<b>b. </b>"
+++

We're ready to compile and flash our application on to the device, provision the Wi-Fi through the ESP Rainmaker Phone App, register a user to assign the device to, and begin controlling our peripherals over AWS IoT.

## Opening the project in PlatformIO
There are several folders at the root of the project you cloned or downloaded and extracted. For this tutorial, we will open the Getting-Started project in PlatformIO. First open Visual Studio Code and after waiting a few seconds for the PlatformIO extension to load, click the **PlatformIO logo** on the left bar, select **Open** from the left PlatformIO menu, click **Open Project**, navigate to the `Core2-for-AWS-IoT-EduKit/Getting-Started` folder, and click **open**.
{{< img "pio_home.png" "PlatformIO home screen" >}}

## Identifying and setting the serial port on host machine
You need to know which port you'll use for communications with the device for uploading firmware or monitoring the serial output. For macOS, the device is typically on **/dev/cu.SLAB_USBtoUART** and the included configuration should not require any changes. Linux is typically **/dev/ttyUSB0** and requires the user be added to the dialout group. For Windows, you'll need to replace the value with **COM** and end with the corresponding port number (e.g. COM3). For more specific Windows instructions or Linux distros that require additional configuration, please reference [Espressif's offical doc for establishing serial connections with the ESP32](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/establish-serial-connection.html). 

Choose the Visual Studio Code file explorer (<i class="far fa-copy"></i> on the left bar), open the **platformio.ini** file, and change your **upload_port** value to match the serial port the device is connected to.

{{% notice note %}}
View the official PlatformIO docs about the [upload_port](https://docs.platformio.org/en/latest/projectconf/section_env_upload.html#upload-port) option to see formatting and other examples.
{{% /notice %}}

## Building and uploading the RainMaker Agent firmware
You are now ready to build and flash your firmware. Start by clicking the PlatformIO logo on the left bar. Then click the <i class="fas fa-redo"></i> icon (hover your mouse next to **PROJECT TASKS**) to refresh the available options. You'll click the **Build** button to start building the device firmware. This will take a few minutes depending on your computer. Once that is complete, click **Upload and Monitor** option from the PlatformIO project tasks menu. Once the upload has completed successfully, the device will boot with the firmware that was just compiled and uploaded. It will also begin display the serial output from the device in that terminal viewport (now the serial monitor window). The device will be going through the process of generating security keys and performing a self claim. Key generation can take a few seconds, up to a few minutes to complete but once claiming completes a QR code will display on the serial monitor window.
{{< img "pio_menu.png" "PlatformIO Menu to build, flash, monitor" >}}

{{% notice note %}}
For additional troubleshooting or FAQs, visit the official [Espressif RainMaker FAQs](https://rainmaker.espressif.com/docs/faqs.html).
{{% /notice %}}

## Claiming and provisioning the device
On your mobile phone, open the ESP RainMaker Phone App, grant the requested mobile app permissions, press **Add Device**, and then scan the QR code displayed on your computer's serial monitor (*not* the QR code on the device). It will then go through the provisioning process, which includes Wi-Fi provisioning with your Wi-Fi credentials for your 2.4GHz wireless home network. After a successful Wi-Fi connection, the device will authenticate itself and your phone app will populate with multiple virtual devices that can be viewed and/or controlled. If the virtual devices are marked **offline** on the phone app after a minute or two, try scrolling down to refresh.
{{< img "qr_code_scan.png" "Scan the QR code in serial output" >}}

With the virtual device listed and online in your mobile app, you can turn the on-board motor or LEDs on or off, adjust the speed of the motor, set the color and brightness of the LED bars, and view the internal device temperature.

{{% notice info %}}
If you entered the wrong Wi-Fi credentials, you'll need to [erase the firmware](/en_uk/getting-started/run-rainmaker.html#erasing-the-firmware-with-platformio) first, re-upload the Espressif RainMaker Agent firmware to the device, and add the device with your mobile phone again.
{{% /notice %}}

## Erasing the firmware with PlatformIO
Once you are done with this application and ready to move on to other tutorials, you'll need to first wipe the device firmware. However, you'll need to stop the active serial monitor first since it's blocking other communication to the port. You can stop the serial monitor by selecting the Visual Studio Code pane with the serial monitor running and pressing **CTRL** + **C**. Then to erase the flash memory, enter the PlatformIO menu, expand the **Platform** menu list, and then click the **Erase Flash** option. This will remove all the data on the device, including the RainMaker certificates that were used. 
{{< img "pio_platform_menu.png" "Erasing flash with PlatformIO" >}}

{{% notice note %}}
Powercycling the device with a empty flash will result in the device screen being blank and a audible ticking sound from the speaker. This is expected behavior as the device is continually rebooting itself without an application to run.
{{% /notice %}}

## Conclusion
You've just built a connected home application through the AWS IoT EduKit program! Not only that, but you have the tools necessary to create, edit, compile, and flash embedded code on to your device! In the next tutorials, you'll get more hands-on and learn the skills to start building your own end-to-end IoT solutions.


On to [**Blinky Hello World**](/en_uk/blinky-hello-world.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://community.m5stack.com/category/41/core2-for-aws" icon="far fa-question-circle" %}}Community support{{% /button %}}