+++
title = "Run the ESP RainMaker Agent"
weight = 12
pre = "<b>b. </b>"
+++

In this section, you compile and upload your application to the AWS IoT EduKit, provision Wi-Fi through the ESP RainMaker Phone App, register a user to assign the devices to, and control the on-board peripherals as virtual smart home devices over AWS IoT.

## Open the project in PlatformIO
There are several folders at the repository root that you cloned from GitHub during the last section. For this tutorial, use the *Getting-Started* project folder and perform the following operations through PIO. 

Complete the following steps to open the project in PIO: 
1. Open the VS Code application. 
1. Choose the **PlatformIO logo** in the VS Code activity bar.
1. Select **Open** from the PIO Home menu. 
1. Choose **Open Project**, navigate to the `Core2-for-AWS-IoT-EduKit/Getting-Started` folder and select **Open Getting-Started**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Expand the PIO menu, 2 - Select Open, 3 - Choose the Getting-Started project folder" >}}

## Build and upload the ESP RainMaker Agent firmware
You are now ready to build (compile) and upload the ESP RainMaker Agent firmware. 

The [GCC compiler](https://gcc.gnu.org/onlinedocs/gcc/) converts the provided human readable code into [object code](https://en.wikipedia.org/wiki/Object_code) as [elf](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) and binary files (the firmware). These files are then uploaded to the device's on-board flash memory for it to execute through the virtual serial port. The serial port (via UART) allows bi-directional communication; so, the AWS IoT EduKit can collect data and sent it to the host machine. 

Complete the following steps to build and upload the firmware to the AWS IoT EduKit. , and monitor the output from the device through the serial port using PlatformIO:
1. Connect the AWS IoT EduKit to your host machine.
1. Choose the **PlatformIO logo** on the VS Code activity bar to open the Quick Access menu.
1. Expand **Miscellaneous** and select **New Terminal**.
1. To start the build process, paste in the command below (this will take several minutes) in the new terminal window that opened in VS Code:
    ```bash
    pio run --environment core2foraws
    ```

{{% notice info %}}
PIO dependencies are installed in the background for the device platform. If those operations do not complete successfully, you might encounter an error. If you do receive an error, wait a couple minutes and run the command again.
{{% /notice %}}

{{< img "pio-new_terminal-gs.en.png" "PlatformIO CLI terminal in VS Code" "1 - Open PIO menu, 2 - Open new PIO Terminal, 3 - Verify you're in the PlatformIO CLI terminal session, 4 - Paste the command into terminal, 5 - If you encounter an error autodetecting the port, open the Platform.ini file and follow instructions to manually add the serial port.">}}

4. Issue the following command to upload the compiled firmware to the connected device through USB:
    ```bash
    pio run --environment core2foraws --target upload
    ```
1. Issue the following command to monitor the serial output from the device on your host machine:
   ```bash
   pio run --environment core2foraws --target monitor
   ```
{{% notice info %}}
If you receive an error that the serial port timed out or there is an incorrect port number, open the `platformio.ini` file, and follow its instructions to manually set the upload port.
{{% /notice %}}
## Claim and provision the device 
Once the upload has successfully completed, the device will boot using the firmware that was just uploaded. It will also display the device's serial output in the terminal viewport. 

The device will generate security keys and perform an [assisted claim](https://rainmaker.espressif.com/docs/claiming.html#assisted-claiming-esp32). Key generation can take up to a few minutes to complete. When the assisted claiming process, a quick response (QR) code displays on the terminal viewport.

Complete the following steps to add your AWS IoT EduKit to the application:
1. Open the ESP RainMaker Phone application on your smartphone.
1. Grant the application's requested permissions and choose **Add Device**.
1. Scan the QR code. 

The application will then go through the provisioning process, which includes establishing a connection to your 2.4GHz wireless home network. After connecting successfully to your home Wi-Fi, the device will authenticate itself and your phone app will populate with multiple virtual devices that can be viewed and controlled. If the ESP RainMaker app shows displays the virtual devices as **offline**, wait a minute or two and pull down the page to refresh the app.
{{< img "pio-qr_code_scan.en.png" "Scan the QR code in serial output" >}}

With the virtual device listed and online in your mobile app, you can turn the on-board motor or LEDs on or off, adjust the speed of the motor, set the color and brightness of the LED bars, and view the internal device temperature.

{{% notice info %}}
If you entered the wrong Wi-Fi credentials, you must [erase the firmware](/en/getting-started/run-rainmaker.html#erasing-the-firmware-with-platformio). To do this, upload the ESP RainMaker Agent firmware to the device a second time and add the device with your mobile phone again. For additional troubleshooting or FAQs, visit the official [ESP RainMaker FAQs](https://rainmaker.espressif.com/docs/faqs.html).
{{% /notice %}}

## Erase the firmware with PlatformIO
Once you complete this process and are ready for the other tutorials, you need stop the active serial monitor and erase the device's firmware. 

Complete the following steps: 
1. Return to VS Code and enter **CTRL+C** in the PIO terminal to stop the serial monitor.
1. Issue the following command to erase the firmware from the device's flash memory:
```bash
pio run --environment core2foraws --target erase
```

>  When the command in the  terminal window completes, the AWS IoT EduKit may not automatically restart or appear to be different. Power the device off and then on to confirm that the firmware was flashed.

{{% notice note %}}
Erasing the firmware results in a blank device screen and an audible ticking sound. This is expected behavior. The device is continually rebooting itself because it doesn't have an application to run.
{{% /notice %}}

## Conclusion
Congratulations! During this lesson, you built a connected home application through the AWS IoT EduKit program. Not only that, but you also have the tools necessary to create, edit, compile, and flash embedded code on to your device! In the next tutorials, you'll get more hands-on practice and will learn the skills to start building your own end-to-end IoT solutions.

On to [**Cloud Connected Blinky**](/en/blinky-hello-world.html)!

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}