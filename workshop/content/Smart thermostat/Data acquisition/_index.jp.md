+++
title = "データ取得"
weight = 20
pre = "<b>b. </b>"
+++

## 本章の概要
この章の終わりには、デバイスで次のようなことができるようになります。

* 電源をオンにしてローカルのスマートサーモスタットアプリケーションを起動する
* 検出された現在の温度、測定された雑音レベル、HVAC の状態 (heating、cooling、standby)、在室状況のインジケーターを使用してログにパブリッシュする

Core for AWS IoT EduKit リファレンスハードウェアキットには、すぐに使用できるセンサーが複数付属しています。このソリューション用に、ローカルのアプリケーションコードを使用した温度センサーとマイクから読み込みを実行します。アプリケーションはセンサーの読み込みと、ディスプレイに概要を表示するため使用される状態のフラグを追跡し続けます。

## サーモスタットアプリケーションのプログラミング方法
スマートサーモスタットは統合されたセンサーをサンプリングします。これには、すでに作成され、バンドル化されたソフトウェアコンポーネントに含まれている実装が使用されます。最初のステップでは、AWS IoT Core までのセンサー値のパブリッシュに移る前に、値をキャプチャしてロガーにプリントアウトするだけです。

含まれている MPU6886 モジュールの温度センサーから読み込むのは簡単です。以下はコードスニペットです。

```c
// include libraries for interfacing with the kit's modules
#include "freertos/FreeRTOS.h"
#include "core2forAWS.h"
#include "mpu6886.h"
#include "esp_log.h"

// the application to run on the device
void app_main()
{
  // initialize a float to store our temperature reading
  float temperature = 0.0f;

  // initialize the kit modules
  Core2ForAWS_Init();
  MPU6886_Init();

  // read from the temperature sensor
  MPU6886_GetTempData(&temperature);

  // convert the reading to Fahrenheit and apply a calibration offset of -50
  temperature = (temperature * 1.8)  + 32 - 50;

  // write the value to the ESP32 logger
  ESP_LOGI("thermostat", "measured temperature is: %f", temperature);
}
```

このコードを使用してデバイスログをビルド、フラッシュ、モニタリングした場合、デバイスは温度センサーから1回読み込みを実行し、ロガーに書き込み、停止します。他のプロセスをブロックすることなく毎秒のサンプルを継続的に取得する場合、個別の **[FreeRTOS タスク](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/system/freertos.html#_CPPv423xTaskCreatePinnedToCore14TaskFunction_tPCKcK8uint32_tPCv11UBaseType_tPC12TaskHandle_tK10BaseType_t)**を作成し、MCU コアにピン付けします。以下を参照してください。

```c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "core2forAWS.h"
#include "mpu6886.h"
#include "esp_log.h"

// store our application logic in a task function
void temperature_task(void *arg) {
  float temperature = 0.0f;

  // loop forever!
  for (;;) {
    MPU6886_GetTempData(&temperature);
    temperature = (temperature * 1.8)  + 32 - 50;
    ESP_LOGI("thermostat", "measured temperature is: %f", temperature);

    // sleep for 1000ms before continuing the loop
    vTaskDelay(1000 / portTICK_RATE_MS);
  }
}

void app_main()
{
  Core2ForAWS_Init();
  MPU6886_Init();

  // FreeRTOS concept: operations that run in a continuous loop are done in tasks
  xTaskCreatePinnedToCore(&temperature_task, "temperature_task", 4096, NULL, 5, NULL, 1);
}
```

## コードサンプル
サンプルアプリケーションはすでに用意され、ビルドしてデバイスにデブロイできるようになっています。これらのステップに沿って、この章を完了してください。

1. 別のチュートリアルで`cloneしていない場合`、このチュートリアル用のコードレポをcloneします
   ```bash
   git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
   ```
2. ディレクトリを**Smart-Thermostat**プロジェクトに変更します
   ```bash
   cd Core2ForAWS-AWS-IoT-EduKit/Smart-Thermostat
   ```
3. **Blinky-Hello-World** デモの `sdkconfig` ファイルをコピーし、Wi-Fi と AWS エンドポイントの設定をします
   ```bash
   cp ../Blinky-Hello-World/sdkconfig .
   ```
   * 任意で、ファームウェア向けに Wi-Fi 認証情報と AWS エンドポイントを手動で設定できます。「**Blinky Hello World(Lチカ)**」の例での「**[ESP32 ファームウェアの設定](/en/blinky-hello-world/connecting-to-aws.html#configuring-the-esp32-firmware)**」のステップに沿って設定します
4. シリアルモニターをビルド、フラッシュ、開始します
   ```bash
   idf.py build flash monitor -p <<DEVICE_PORT>> 
   ```
   **<<DEVICE_PORT>>** は、デバイスが接続されているポートに合わせてください。詳しくはBlinky Hello World(Lチカ)の[ホストマシンのシリアルポートの特定](/ja/blinky-hello-world/device-provisioning.html)を参照してください
5. アプリをビルドしてフラッシュするのには時間がかかりますが、完了したらターミナルにデバイスログのストリームが表示されるようになります。**Ctrl** + **]** を入力すると、モニターセッションを閉じることができます

{{% notice note %}}
シェルプロンプトに **(edukit)** が表示されていない場合は、`conda activate edukit` を入力して、conda環境を起動します。
`idf.py`コマンドが見つからない場合は、`. $HOME/esp/esp-idf/export.sh` (macOS/Linux) or `%userprofile%\Desktop\esp-idf\export.bat` (Windows) のコマンドを実行して、パスにESP-IDFを追加します
{{% /notice %}}

## 検証ステップ
次の章に進む前に、デバイスが想定どおりに設定されているかを検証できます。

1. Core2 for AWS IoT EduKit の電源がオンになっていてアプリケーションが実行されている場合、ターミナルウィンドウのログに次のような表示が確認できます

```
I (16128) shadow: On Device: roomOccupancy false
I (16132) shadow: On Device: hvacStatus STANDBY
I (16137) shadow: On Device: temperature 64.057533
I (16143) shadow: On Device: sound 8
```

想定どおりに機能している場合は、「[データ同期](/ja/smart-thermostat/data-sync.html)」に進みましょう

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}