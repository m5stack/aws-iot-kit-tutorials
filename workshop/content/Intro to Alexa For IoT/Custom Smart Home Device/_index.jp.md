+++
title = "カスタムスマートホームデバイス"
weight = 40
pre = "<b>d. </b>"
+++

## スマートホームコントロールのカスタマイズ
キットにどのようなスマートホームの制御機能が含まれているかを理解したところで、次は、それらの機能を修正して、単にシリアルモニターに印刷するというよりもむしろ、デバイス自体の属性を制御します。このワークショップでは、**PowerController** を介して側面の緑色の LED ライトを点灯させる簡単な実装を作成し、Range Controller を使って設定速度でデバイスが点滅するように設定します。

## スマートホームデバイスの属性のカスタマイズ
IDE を使用して、cloneを作成した **Core2-for-AWS-IoT-EduKit** フォルダを開き、`Alexa_for_IoT-Intro/components/app_smart_home/app_smart_home.c` を開いて、スマートホームデバイスの属性が定義されていることを確認します。109 行の **app_smart_home_init()** 関数までスクロールダウンすると、以下のコードブロックが表示されます:
```c
/* Add device */
smart_home_device_t *device = smart_home_device_create("Light", alexa_smart_home_get_device_type_str(LIGHT), NULL);
smart_home_device_add_cb(device, write_cb, NULL);
smart_home_node_add_device(node, device);
```

上記のブロックはデバイスの「フレンドリ名」、つまりデバイスの名前を定義します。**Light** を **Green Light** に変更することで、名前がより明確になります。2 行目は以下のようになります。

`smart_home_device_t *device = smart_home_device_create("Green Light", alexa_smart_home_get_device_type_str(LIGHT), NULL);`

次に、Range Controller の定義を変更して、**Brightness** を **Blink** に変更します。もう少し下の141 行までスクロールダウンすると、デバイスのさまざまな属性が定義されているのが確認できます。
```c
/* Add device parameters */
smart_home_param_t *power_param = smart_home_param_create("Power", SMART_HOME_PARAM_POWER, smart_home_bool(true), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_device_add_param(device, power_param);

smart_home_param_t *brightness_param = smart_home_param_create("Brightness", SMART_HOME_PARAM_RANGE, smart_home_int(100), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(100), smart_home_int(1));
smart_home_device_add_param(device, brightness_param);
```

点滅回数の最大値を100から10に更新する必要があります。そこで、**brightness_param** 変数の定義と **smart_home_param_add_bounds** 関数の呼び出しに変更を加えると、以下のように表示されます。
```c
smart_home_param_t *brightness_param = smart_home_param_create("Blink", SMART_HOME_PARAM_RANGE, smart_home_int(10), SMART_HOME_PROP_FLAG_READ | SMART_HOME_PROP_FLAG_WRITE | SMART_HOME_PROP_FLAG_PERSIST);
smart_home_param_add_bounds(brightness_param, smart_home_int(0), smart_home_int(10), smart_home_int(1));
```

## スマートホームデバイスロジックの追加

ここまででプロパティを変更しましたが、緑色の LED を制御するロジックを実装する必要があります。まず、ライトを制御するためのライブラリを組み込んで、いくつかのグローバル変数も定義する必要があります:
```c
#include "axp192.h"
static uint8_t CURRENT_BLINK_DELAY = 20;
static uint8_t GREEN_LIGHT_STATUS = 0;
```

次に、パワーコントローラーの実装を変更して、緑色の LED を点滅・消灯させます。 **write_cb** 関数 (元のファイルの 89 行目) Aと次のコードスニペットを探します。
```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");

}
```

受信した値に基づいたトグルロジックを追加するには、if ステートメントが次のようになるように、3 行のコードを追加します
```c
if (val.type == SMART_HOME_VAL_TYPE_BOOLEAN) {
    printf("%s: *************** %s's %s turned %s ***************\n", TAG, device_name, param_name, val.val.b ? "ON" : "OFF");
    
    //Set the global GREEN_LIGHT_STATUS variable to the desired value and set the GPIO1 value the right setting (on/off)
    GREEN_LIGHT_STATUS = val.val.b ? 0 : 1;
    Axp192_SetGPIO1Mode(GREEN_LIGHT_STATUS);

}
```

顧客がデバイスをオンにしたいかオフにしたいかを示す電源コントローラーの更新値 (1 または 0) を取得し、側面の緑色のライトをオンまたはオフにします。

次に、「点滅」機能を実装します。まず、設定された回数だけ緑色のライトを点滅させるメソッドを作成する必要があります。次の関数を同じ **app_smart_home.c** ファイルの **void app_device_driver_init()** 関数の前に追加します。
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

この関数を呼び出す RangeController ロジックを更新しします。この場合、**write_cb** の次のコードブロックにあります。
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

}
```
新しく作成した startLinkBlink 関数の呼び出しを追加するだけです。ここに以下のように行を追加します。
```c
else if (val.type == SMART_HOME_VAL_TYPE_INTEGER) {
    printf("%s: *************** %s's %s changed to %d ***************\n", TAG, device_name, param_name, val.val.i);

    //call the Trigger Light Blink based on the desired number of actions
    startLightBlink(val.val.i);        

}
```

## アップデートされた Alexa ファームウェアのフラッシュとテスト
必要なデバイス属性と制御ロジックを持つようにプロジェクトが変更され、オンボードの緑の LED が指定の速度で点滅します。ファームウェアを構築し、リファレンスハードウェアにフラッシュし、機能をテストします。デバイスを接続し、PlatformIO CLI[ターミナルウィンドウ](/jp/blinky-hello-world/prerequisites.html#platformio)を開いて選択し、以下のコマンドを貼り付けます。
```bash
pio run --environment core2foraws --target upload --target monitor
```
{{% notice info %}}
**CTRL** + **C** キーを押すと、シリアルモニターから抜けます。コマンド `pio run --environment core2foraws --target monitor` を入力すると、シリアルモニターを再起動することができます。
{{% /notice %}}

すべてがうまくいくと、**Green Light** という名前の新しいデバイスが追加されたことを示す更新情報が Alexa アプリに表示されます。電源を制御してみましょう。
!["Green Light" device added notification](custom-smart-home-device/alexa_app-green_light-found.en.jpg?height=500px&classes=shadow)

![Device named "Green Light" listed in your Alexa App](custom-smart-home-device/AlexaApp-GreenLight.png?height=500px&classes=shadow)

* 音声: _Alexa, turn on/off green light light_  - 側面の緑色の LED ライトがオフまたはオンになります
* Alexa アプリ経由の場合 - Alexa モバイル (Espressif アプリではありません) を開き、[デバイス] に進み、次に [Lights] または [All Devices] のどちらかに進むと、[Green Light] というデバイスが表示されます (下のスクリーンショットを参照してください)。電源アイコンをタップすると、アイコンのオフとオンが切り替わることがわかります。

{{< img "alexa_app-green_light-power_press.webp" "Power Controller implemented with the green light" >}}

点滅機能をテストすることもできます。

音声: _Alexa, set blink to 7_ に設定して - 側面の緑色の LED ライトが 7 回点滅します
Alexa アプリ経由 - Alexa アプリの **Green Light** デバイス (ESP Alexa スマートフォンアプリではありません) から、1～10 の間でスライダーを調整すると、指定した回数だけデバイスが点滅します。

{{< img "alexa_app-green_light-blink_slider.en.webp" "Blinking the green LED with power controller slider" >}}

おめでとうございます。このチュートリアルが完了しました。 [まとめ](/jp/intro-to-alexa-for-iot/conclusion.html)をご覧ください。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}