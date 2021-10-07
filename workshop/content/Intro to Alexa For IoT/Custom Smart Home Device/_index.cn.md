+++
title = "自定义智能家居设备"
weight = 40
pre = "<b>d. </b>"
+++

## 自定义智能家居控制功能
现在我们了解了该套件包含哪些智能家居控制功能，接下来您可以修改这些功能来控制设备本身的属性，而不是只输出到串行监控器。在此研讨会中，您将通过 **PowerController** 创建一个能打开绿色 LED 侧灯的简单实现，然后使用范围控制器将设备设置为以固定速率闪烁。

## 自定义智能家居设备属性
使用您的 IDE 打开克隆的 **Core2-for-AWS-IoT-EduKit** 文件夹，然后打开 `Alexa_for_IoT-Intro/esp-va-sdk-core2foraws/examples/additional_components/app_smart_home/app_smart_home.c`，以查看智能家居设备属性是在哪里定义的。向下滚动到第 109 行的 **app_smart_home_init()** 函数，您会发现以下代码块：
```c
/* Add device */
smart_home_device_t *device = smart_home_device_create("Light", alexa_smart_home_get_device_type_str(LIGHT), NULL);
smart_home_device_add_cb(device, write_cb, NULL);
smart_home_node_add_device(node, device);
```
上述代码块定义了设备的 “友好名称”，也就是设备的名称。将 **Light** 更改为 **Green Light** 可让名称更明确。第二行应类似于以下内容：

`smart_home_device_t *device = smart_home_device_create("Green Light", alexa_smart_home_get_device_type_str(LIGHT), NULL);`

接着，修改范围控制器定义，将 **Brightness** 更改为 **Blink**。再向下滚动到第 141 行，您可以看到设备的各种属性是在哪里定义的：
```c
/* Add device parameters */
smart_home_param_t *power_param = smart_home_param_create("Power", SMART_HOME_PARAM_POWER, smart_home_bool(true), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_device_add_param(device, power_param);

smart_home_param_t *brightness_param = smart_home_param_create("Brightness", SMART_HOME_PARAM_RANGE, smart_home_int(100), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(100), smart_home_int(1));
smart_home_device_add_param(device, brightness_param);
```

您需要将最大闪烁次数从 `100` 更新到 `10`。因此您需要更改 **brightness_param** 变量定义和 **smart_home_param_add_bounds** 函数调用，应类似于以下内容：
```c
smart_home_param_t *brightness_param = smart_home_param_create("Blink", SMART_HOME_PARAM_RANGE, smart_home_int(10), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(10), smart_home_int(1));
```

## 添加智能家居设备逻辑
您已经修改了属性，但还需要实施逻辑来控制绿色 LED。首先，我们需要添加一个用于控制灯的库，并定义几个全局变量：
```c
#include "core2forAWS.h"
#define BLINK_DELAY 500
static bool green_light_status = 0;
```

接着，修改电源控制器来打开/关闭绿色 LED。查找 **write_cb** 函数（源文件的第 89 行）和以下代码段：
```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");

}
```

接下来，通过添加三行代码来添加基于接收的值的切换逻辑，if 语句类似于以下内容：
```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");
    
    //Set the global green_light_status variable to the desired value and set the GPIO1 value the right setting (on/off)
    green_light_status = val.val.b ? 0 : 1;
    Core2ForAWS_LED_Enable(green_light_status);

}
```

您将使用电源控制器的更新值（1 或 0）来表示客户是否希望打开设备以及侧面的绿灯。

接着实施 “闪烁” 功能。首先，您需要创建一个让绿灯闪烁固定次数的方法。将以下函数添加到该 **app_smart_home.c** 文件中：
```c
static void startLightBlink(int count)
{    
    // Toggle the LED to the opposite of the LED state and get back to the original LED state at the end
    for(int i = (1 + green_light_status); i <= (count * 2 + green_light_status); i++) {
        Core2ForAWS_LED_Enable( i % 2 );
        vTaskDelay(pdMS_TO_TICKS(BLINK_DELAY));
    }
}
```

现在更新 RangeController 逻辑以调用该函数。在这种情况下，逻辑是 **write_cb** 后的下列代码块：
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

}
```
我们需要增加到新创建的 *startLinkBlink* 函数的调用，添加下列行：
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

    //call the Trigger Light Blink based on the desired number of actions
    startLightBlink(val.val.i);        

}
```

## 烧录并测试更新的 Alexa 固件
您已经修改了项目，有了让板载绿色 LED 以指定速率闪烁所需的设备属性和控制逻辑，下面构建固件、将固件烧录到参考硬件上并测试功能。将设备连接上，在您的 [PlatformIO CLI 终端窗口](../blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window) 中，打开，选择并输入下面的命令：
```bash
pio run --environment core2foraws --target upload --target monitor 
```
{{% notice info %}}
您可以使用 **CTRL** + **C** 退出串口监视器。您可以通过使用命令 `pio run --environment core2foraws --target monitor ` 重启串口监视器。
{{% /notice %}}

如果一切顺利，您的 Alexa 应用程序中将显示一个更新，您将看到名为 **Green Light** 的新设备。尝试控制电源：
!["Green Light" device added notification](custom-smart-home-device/alexa_app-green_light-found.en.jpg?height=500px&classes=shadow)

![Device named "Green Light" listed in your Alexa App](custom-smart-home-device/alexa_app-green_light-power_on.en.png?height=500px&classes=shadow)

* 语音: _Alexa, turn on/off green light light_ - 这台绿色的 LED 侧灯就会打开或关上。
* 通过 Alexa app - 打开您的 Alexa app（而不是 Espressif app），转至 Devices（设备），然后转至 “Lights（灯）” 或 “All Devices（所有设备）”，您就可以看到名为 “Green Light” 的设备（参见以下屏幕截图）。点击电源图标，您应该会看到开关标示。

{{< img "alexa_app-green_light-power_press.webp" "Power Controller implemented with the green light" >}}

您还可以测试闪烁功能：

* 语音: _Alexa, set blink to 7_ - 绿色的 LED 侧灯就会闪烁 7 次
* 通过 Alexa app - 在 Alexa app（而不是 ESP Alexa app）的 **Green Light** 设备中调整滑块（1 到 10），设备就会闪烁指定次数。
{{< img "alexa_app-green_light-blink_slider.en.webp" "Blinking the green LED with power controller slider" >}}

恭喜，您已学完此教程！进入 [**总结**](/cn/intro-to-alexa-for-iot/conclusion.html) 部分。

---
{{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}} {{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}}