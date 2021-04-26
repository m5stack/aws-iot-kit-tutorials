+++
title = "Running the ESP RainMaker Agent"
weight = 12
pre = "<b>b. </b>"
+++

We're ready to compile and upload our application on to the device, provision the Wi-Fi through the ESP RainMaker Phone App, register a user to assign the devices to, and begin controlling the on-board peripherals as virtual smart home devices over AWS IoT.

## Opening the project in PlatformIO
There are several folders at the root of the repository you cloned from GitHub in the last chapter. For this tutorial, you will use the Getting-Started project and perform the operations you need in PlatformIO. First open Visual Studio Code and after waiting a few seconds for the PlatformIO extension to load, click the **PlatformIO logo** VS Code activity bar (left most menu), select **Open** from the left PlatformIO menu, click **Open Project**, navigate to the `Core2-for-AWS-IoT-EduKit/Getting-Started` folder, and click **open**.
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - Open PIO menu, 2 - Open PIO home, 3 - Open Getting Started project" >}}

## Building and uploading the RainMaker Agent firmware
You are now ready to build (compile) and upload the RainMaker Agent firmware. The compiling is done by the [GCC compiler](https://gcc.gnu.org/onlinedocs/gcc/), which converts the provided human readable code into [object code](https://en.wikipedia.org/wiki/Object_code) as [elf](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) and binary files (the firmware). These files are then uploaded to the device's on-board flash for it to execute via the virtual serial port. The serial port (via UART) allows bi-directional communication, so you are able to recieve data from the device to the host machine. To build and then upload the firmware, and monitor the output from the device through the serial port using PlatformIO:
1) Click the **PlatformIO logo** on the VS Code activity bar (left most menu).
2) From the **Quick Access** menu, under **Miscellaneous**, select **New Terminal**.
3) To start the build process, paste in the command below (this will take several minutes) in the new terminal window that opened in VS Code:
    ```bash
    pio run --environment core2foraws
    ```
{{< img "pio-new_terminal-gs.en.png" "PlatformIO CLI terminal in VS Code" "1 - Open PIO menu, 2 - Open new PIO Terminal, 3 - Verify you're in the 'PlatformIO CLI' terminal session, 4 - Paste the command into terminal, 5 - If you encounter an error autodetecting the port, open the Platform.ini file and follow instructions to manually add the serial port.">}}

4) Now it's time to upload the compiled firmware to the connected device over USB by running the command:
    ```bash
    pio run --environment core2foraws --target upload
    ```
5) Lastly, monitor the serial output from the device on your host machine via:
   ```bash
   pio run --environment core2foraws --target monitor
   ```
{{% notice info %}}
If during upload or monitoring the serial output you receive an error about an incorrect ports or timeout, open the `platformio.ini` file, and follow the instructions in that file for how to manually set the upload port.
{{% /notice %}}
## Claiming and provisioning the device
Once the upload has completed successfully, the device will boot with the firmware that was just compiled and uploaded. It will also display the serial output from the device in that terminal viewport. The device will be going through the process of generating security keys and performing an [assisted claim](https://rainmaker.espressif.com/docs/claiming.html#assisted-claiming-esp32). Key generation can take a few seconds, up to a few minutes to complete but once claiming completes a QR code will display on the terminal viewport.

On your mobile phone, open the ESP RainMaker Phone App, grant the requested mobile app permissions, press **Add Device**, and then scan the QR code ouput via the serial monitor in the terminal viewport. It will then go through the provisioning process, which includes Wi-Fi provisioning with your Wi-Fi credentials for your 2.4GHz wireless home network. After a successful Wi-Fi connection, the device will authenticate itself and your phone app will populate with multiple virtual devices that can be viewed and/or controlled. If the virtual devices are marked **offline** on the phone app after a minute or two, try scrolling down to refresh.
{{< img "pio-qr_code_scan.en.png" "Scan the QR code in serial output" >}}

With the virtual device listed and online in your mobile app, you can turn the on-board motor or LEDs on or off, adjust the speed of the motor, set the color and brightness of the LED bars, and view the internal device temperature.

{{% notice info %}}
If you entered the wrong Wi-Fi credentials, you'll need to [erase the firmware](/en/getting-started/run-rainmaker.html#erasing-the-firmware-with-platformio) first, re-upload the Espressif RainMaker Agent firmware to the device, and add the device with your mobile phone again. For additional troubleshooting or FAQs, visit the official [Espressif RainMaker FAQs](https://rainmaker.espressif.com/docs/faqs.html).
{{% /notice %}}

## Erasing the firmware with PlatformIO
Once you are done with this application and ready to move on to other tutorials, you'll need to first erase the device firmware. However, you'll need to stop the active serial monitor first since it's blocking the virtual serial communications port. You can stop the serial monitor by pressing **CTRL** + **C**. To erase the firmware from the device's flash memory, you enter the command:
```bash
pio run --environment core2foraws --target erase
```

{{% notice note %}}
Powercycling the device with a empty flash will result in the device screen being blank and a audible ticking sound from the speaker. This is expected behavior as the device is continually rebooting itself without an application to run.
{{% /notice %}}

## Conclusion
You've just built a connected home application through the AWS IoT EduKit program! Not only that, but you have the tools necessary to create, edit, compile, and flash embedded code on to your device! In the next tutorials, you'll get more hands-on and learn the skills to start building your own end-to-end IoT solutions.

On to [**Blinky Hello World**](/en/blinky-hello-world.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}