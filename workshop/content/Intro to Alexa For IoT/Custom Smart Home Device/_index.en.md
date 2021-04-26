+++
title = "Custom Smart Home Device"
weight = 40
pre = "<b>d. </b>"
+++

## Customizing Smart Home Control
Now that we have an understanding of what Smart Home control capabilities are included with the kit, you will modify those capabilities to control attributes of the device itself versus simply printing to the serial monitor. For this workshop, you will create a simple implementation that turns on the side green LED light via the **PowerController** and we will set the device to blink at a set rate using the Range Controller.

## Customizing smart home device attributes
Using your IDE, open the cloned **Core2-for-AWS-IoT-EduKit** folder and open the `Alexa_for_IoT-Intro/components/app_smart_home/app_smart_home.c` to take a look at where the Smart Home device attributes are defined. Scrolling down to line 109 **app_smart_home_init()** function and you will see the following code blocks:
```c
/* Add device */
smart_home_device_t *device = smart_home_device_create("Light", alexa_smart_home_get_device_type_str(LIGHT), NULL);
smart_home_device_add_cb(device, write_cb, NULL);
smart_home_node_add_device(node, device);
```
The above block defines the "friendly name" of the device, that is - the name of the device. Be more explicit with the name by changing **Light** to **Green Light**. The second line should look like the following:

`smart_home_device_t *device = smart_home_device_create("Green Light", alexa_smart_home_get_device_type_str(LIGHT), NULL);`

Next, modify the Range Controller definition to change **Brightness** to **Blink**. Scrolling down a bit further to line 141, you can see where the different attributes of our device is defined:
```c
/* Add device parameters */
smart_home_param_t *power_param = smart_home_param_create("Power", SMART_HOME_PARAM_POWER, smart_home_bool(true), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_device_add_param(device, power_param);

smart_home_param_t *brightness_param = smart_home_param_create("Brightness", SMART_HOME_PARAM_RANGE, smart_home_int(100), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(100), smart_home_int(1));
smart_home_device_add_param(device, brightness_param);
```

You will need to update the max blink count from 100 to 10. So you will make a change to the **brightness_param** variable definition and to the **smart_home_param_add_bounds** function call, so it should now look like:
```c
smart_home_param_t *brightness_param = smart_home_param_create("Blink", SMART_HOME_PARAM_RANGE, smart_home_int(10), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(10), smart_home_int(1));
```

## Adding smart home device logic
By now you've modified the properties but need to implement the logic to control the green LED. First, we need to include a library for controlling the light and also define a few global variables:
```c
#include "axp192.h"
static uint8_t CURRENT_BLINK_DELAY = 20;
static uint8_t GREEN_LIGHT_STATUS = 0;
```

Next, modify the power controller implementation to turn on/off the green LED. Look for the **write_cb** function (line 89 of the original file) and the following code snippet:
```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");

}
```

Now to add toggle logic based on the values received by adding three lines of code so that the if statement looks like the following:
```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");
    
    //Set the global GREEN_LIGHT_STATUS variable to the desired value and set the GPIO1 value the right setting (on/off)
    GREEN_LIGHT_STATUS = val.val.b ? 0 : 1;
    Axp192_SetGPIO1Mode(GREEN_LIGHT_STATUS);

}
```

You are taking the updated value of the power controller (1 or 0) to indicate whether the customer wanted to turn the device on or off, and will turn the side green light on or off.

Next, implement the "blink" functionality. First, you'll need to create a method to blink the green light a set number of times. Add the following function to the same **app_smart_home.c** file:
```c
static void startLightBlink(int count)
{    
    //we want to start toggling based on the opposite status of what the light currently is
    for(int i = 1 + GREEN_LIGHT_STATUS; i <= count*2+GREEN_LIGHT_STATUS ; i++) {               
        Axp192_SetGPIO1Mode( i % 2 );
        vTaskDelay(CURRENT_BLINK_DELAY);
    }
}
```

Now update the RangeController logic to call this function. In this case it's in the following code block back in **write_cb**:
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

}
```
We just need to add a call to our newly created *startLinkBlink* function, so we can add the following lines here:
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

    //call the Trigger Light Blink based on the desired number of actions
    startLightBlink(val.val.i);        

}
```

## Flashing and testing updated Alexa firmware
With the project modified to have the necessary device attributes and the control logic to blink the onboard green LED at a specified rate, it is time to build the firmware, flash it on to the reference hardware, and test the capabilities. With the device plugged in and your [PlatformIO CLI terminal window](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window) open and selected, paste familiar the command below:
```bash
pio run --environment core2foraws --target upload --target monitor 
```
{{% notice info %}}
You can exist the serial monitor with the key combination **CTRL** + **C**. You can restart the serial monitor by entering the command `pio run --environment core2foraws --target monitor `.
{{% /notice %}}

If all goes well, you will see an update in your Alexa app that you have a new device called **Green Light**. Try controlling the Power:
!["Green Light" device added notification](custom-smart-home-device/alexa_app-green_light-found.en.jpg?height=500px&classes=shadow)

![Device named "Green Light" listed in your Alexa App](custom-smart-home-device/AlexaApp-GreenLight.png?height=500px&classes=shadow)

* Voice: _Alexa, turn on/off green light light_ - the side green LED light should turn off and on
* Via the Alexa app - open your Alexa app (not the Espressif app), go to Devices and then either "Lights" or "All Devices" and you should see the device named "Green Light" (see screenshots below). Tap the power icon and you should see the icon toggle between off and on. 

{{< img "alexa_app-green_light-power_press.webp" "Power Controller implemented with the green light" >}}

You can also test the Blink functionality:

* Voice: _Alexa, set blink to 7_ - the side green LED light should blink 7 times
* Via the Alexa app - From the **Green Light** device in your Alexa app (not the ESP Alexa mobile phone app), adjust the slider between 1 and 10 and the device should blink the specified number of times.
{{< img "alexa_app-green_light-blink_slider.en.webp" "Blinking the green LED with power controller slider" >}}

Congratulations, you've completed this tutorial! On to the [**Conclusion**](/en/intro-to-alexa-for-iot/conclusion.html).

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}