+++
title = "データ同期"
weight = 30
pre = "<b>c. </b>"
+++

## 本章の概要
この章の終わりには、Core2 for AWS IoT EduKit リファレンスハードウェアキットで、次のことができるようになります

* サンプリングされた環境雑音レベルと部屋の温度を 10 秒ごとに AWS IoT Core にパブリッシュ
* AWS IoT Core の Device Shadow にサブスクライブし、新しいコマンドを取得する

## メッセージの概念
この章では、デバイスをカスタムの AWS IoT Core エンドポイントに接続してから、パブリッシュ/サブスクライブまたは「PubSub」と呼ばれるモデルを用いて、デバイスとクラウド間でメッセージを交換します。 AWS IoT Core は [Message Queueing Telemetry Transport (MQTT)](https://mqtt.org/) というプロトコルに基づいています。ホームページより:

> MQTT はモノのインターネット (IoT、Internet of Things) 向けの OASIS 標準メッセージングプロトコルです。これは非常に軽量のパブリッシュ/サブスクライブ メッセージング トランスポートで、小さいコードフットプリントと最小限のネットワーク帯域幅を有するリモートデバイスの接続に適しています。現在 MQTT は、自動車、製造、電気通信、石油とガスなど、さまざまな業界で使用されています。 

デバイスとクラウド間で交換されるメッセージはテキスト、数字、バイナリ、JSON などさまざまです。JSON は疎結合システム間でのメッセージ交換のベストプラクティスで、AWS IoT サービスにおいて推奨されているパターンです。

メッセージはトピックでパブリッシュされます。トピックはテキストの文字列で、AWS IoT Core の他のクライアントはパブリッシュ メッセージを受信するためにサブスクライブすることができます。トピックに関しては次の例をご覧ください。

* `this is a topic`
* `foo/bar`
* `domain/stuff/thing/whatever`
* `dt/device123/temperature`
* `tokyo/weather/report`
* `$aws/things/edukit/shadow/update`

上記のように、トピックは任意のテキストです。MQTT プロトコルはトピックパーティションを「/ (スラッシュ)」で区切って定義します。これは IoT ソリューションのトピックアーキテクチャ設計の基礎となる概念です。このモジュールは、ベストプラクティスに沿ったトピックの実装をサポートします。

スマートサーモスタットソリューションでは、Device Shadow と呼ばれる AWS IoT Core の機能を使用します。Device Shadow とは、デバイスの状態情報の保存と取得に使用される JSON ドキュメントです。Key-Value ペアを JSON ドキュメントに保存し、特定のトピックに関するドキュメントをパブリッシュすることができます。AWS IoT Core はクラウドにこのドキュメントを保存します。クラウド側からデバイスに対して状態を指示したい場合はこのドキュメントを更新することで、デバイスがオンラインであればリアルタイムに、オフラインであればオンラインになった時に変更を受信できます。

例えば、スマートサーモスタットは最新の部屋の温度と環境雑音レベルを通知し、ソリューションを動作させる必要があります。デバイスによってパブリッシュされたメッセージは次のようになります。

```JSON
{
  "state": {
    "reported": {
      "temperature": 68,
      "noise": 10
    }
  }
}
```

AWS IoT Core はこのようなメッセージを受信すると、新しいメッセージによって上書きされるまで、これらの Key-Value ペアを保存します。

コマンドのスマートサーモスタットへの送信を示すため、他のシステムはこのようなメッセージを Device Shadow にパブリッシュすることができます。

```JSON
{
  "state": {
    "desired": {
      "hvacStatus": "COOLING"
    }
  }
}
```

AWS IoT Core は JSON ドキュメントをマージし、Device Shadow の状態は次のようになります。

```JSON
{
  "state": {
    "reported": {
      "temperature": 68,
      "noise": 10
    },
    "desired": {
      "hvacStatus": "COOLING"
    }
  }
}
```

これは、デバイスが次に AWS IoT Core に接続するか、すでに接続されている場合は、このドキュメントの変更を**desired**として受信します。デバイスコードによって、新しい Key-Value ペア `"hvacStatus": "COOLING"` に反応するかどうかは変わりますが、AWS IoT Core を使用すると、新しい値の通知とコマンドの受信処理が容易になります。

スマートサーモスタットは、前の章で確認したとおり、最新の温度と雑音レベルの値を通知します。サーモスタットはコマンドを受信し、「hvacStatus」と「roomOccupancy」という 2 つの値の状態も追跡するようになります。これらの値は、この後の章で説明するクラウドアプリケーションによって決定されます。

## AWS IoT Core へのメッセージパブリッシュのプログラミング方法
AWS IoT Device SDK for Embedded C (C-SDK) を使用して、スマートサーモスタットデバイスと AWS IoT Core 間の通信を実現します。これはセキュリティ、ネットワーク、データレイヤーを抽象化し、デバイスとソリューションのアプリケーションロジックに焦点を当てられるようにするためのベストプラクティスです。C-SDK は、MQTT プロトコル経由での AWS IoT への接続、ハードウェアセキュアエレメントを使用した署名リクエストへのインターフェイス、Device Shadow の利用などのライブラリをバンドル化します。

コードの主要な行をいくつか確認し、動作を分析しましょう。

```C
jsonStruct_t temperatureHandler;
temperatureHandler.cb = NULL;
temperatureHandler.pKey = "temperature";
temperatureHandler.pData = &temperature;
temperatureHandler.type = SHADOW_JSON_FLOAT;
temperatureHandler.dataLength = sizeof(float);
```

上記のコードは、新しい `jsonStruct_t` を定義します。これは個々の Key-Value ペアの情報を保持するためのに使用され、IoT Core Device Shadow での使用を可能にし、新しく「desierd」メッセージが該当の **pKey** から到着した場合に使用するコールバック機能を示します。この例は新しい Device Shadow キー **temperature** を定義し、`temperature` 変数の値を **pData** の初期値に設定して、`SHADOW_JSON_FLOAT` タイプであることを示します。これらの構造は、後ほどデバイスシャドウの **delta** に対しての、**reported** の値を IoT Core にパブリッシュする際に使用されます。

```C
rc = aws_iot_shadow_register_delta(&iotCoreClient, &roomOccupancyActuator);
```

上記の例では、AWS IoT Device SDK でデルタ動作を登録します。リファレンスを、初期化された SDK クライアントと、変更を適用したいと考えている Key-Value ペアを表す `jsonStruct_t` に渡します。例では、**roomOccupancyActuator** を登録し、クライアントが `state.desired.roomOccupancy` の新しい値を含む新たなシャドウのアップデートをクラウドから受け取ったとき、アクチュエータで定義されたコールバック機能が処理されます。

```C
MPU6886_GetTempData(&temperature);
temperature = (temperature * 1.8)  + 32 - 50;
```

上記は、ローカルの温度読み込みを取得するため MPU6886 コンポーネントから読み込む内容の例です。華氏への変換とハードコードのキャリブレーションオフセットに注意してください。この式は、摂氏で動作するよう変更できます。または、このデフォルトの 50 が期待する範囲での値を生成しない場合、異なるオフセットで動作するよう変更できます。

```C
rc = aws_iot_shadow_add_reported(JsonDocumentBuffer,          
  sizeOfJsonDocumentBuffer, 4, &temperatureHandler,
  &soundHandler, &roomOccupancyActuator, &hvacStatusActuator);
```

このコードには、IoT Core シャドウサービスによって予測されるシャドウドキュメントで、クラウドにパブリッシュする値がすべて含まれています。この変数の関数を変更すると、`state.reported` の Key-Value ペアを追加または削除できます。リストの Key-Value ペアの数に一致するよう、3 つ目のパラメータを必ず更新してください。

```C
rc = aws_iot_shadow_update(&iotCoreClient, 
  CLIENT_ID, JsonDocumentBuffer,
  ShadowUpdateStatusCallback, NULL, 4, true);
```

このコードは、集められたシャドウドキュメントを、トピック `$aws/things/<<CLIENT_ID>>/shadow/update` にて、ネットワーク経由のペイロードとして IoT Core にパブリッシュします。**<<CLIENT_ID>>** の定義は、クライアント ID または、デバイス画面とシリアルモニター出力に表示された Core2 for AWS のシリアル番号です。

```C
while(NETWORK_ATTEMPTING_RECONNECT == rc || NETWORK_RECONNECTED == rc || SUCCESS == rc) {
...
  vTaskDelay(1000 / portTICK_RATE_MS);
}
```

この **xTask** の `while` ループは、ネットワーク接続が切断されない限り、効率的かつ永続的に実行されます。最新のメッセージを Device Shadow サービスにパブリッシュし、受信したメッセージからのdelta コールバックに応答した後、タスクは示されている期間スリープモードに入り、それから次のループのサイクルを継続します。`vTaskDelay()` 式を変更すると、より頻繁に、またはより大きい間隔を空けてパブリッシュすることができます。実際のデプロイをテストして 10、30、または 60 秒に減らす際、1,000 (1 秒) など、より迅速なインターバルの使用は作業に役立つ可能性があります。

```C
void hvac_Callback(const char *pJsonString, uint32_t JsonStringDataLen, jsonStruct_t *pContext) {
    IOT_UNUSED(pJsonString);
    IOT_UNUSED(JsonStringDataLen);

    char * status = (char *) (pContext->pData);

    if(pContext != NULL) {
        ESP_LOGI(TAG, "Delta - hvacStatus state changed to %s", status);
    }

    if(strcmp(status, HEATING) == 0) {
        ESP_LOGI(TAG, "setting side LEDs to red");
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0xFF0000);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0xFF0000);
        Core2ForAWS_Sk6812_Show();
    } else if(strcmp(status, COOLING) == 0) {
        ESP_LOGI(TAG, "setting side LEDs to blue");
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0x0000FF);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0x0000FF);
        Core2ForAWS_Sk6812_Show();
    } else {
        ESP_LOGI(TAG, "clearing side LEDs");
        Core2ForAWS_Sk6812_Clear();
        Core2ForAWS_Sk6812_Show();
    }
}
```

このコードは **hvacStatus** アクチュエータのコールバック機能を示します。これは、新しいメッセージがデバイスによって受信されたときに実行されるコードで、`state.desired.hvacStatus` Key-Value ペアが含まれています。注: 希望する状態の変更を承認するために、何か作業を行う必要はありません。自動的に、ローカルの **hvacStatusActuator** とその **pData** エレメントに適用されます。

`IOTUNUSED()` 関数は、使用されていないパラメータでのコンパイラ警告を制御するため使用されます。

if/else ブロックは、新しい **hvacStatus** Key-Value のテキスト値を評価するため使用され、その情報を LEDの色特定に使用します。

## シリアルコンソールのモニタリング
前章でシリアルモニターを **CTRL** + **C** で終了したり、デバイスの接続を解除した場合、再度シリアルモニターを開始する必要があります。[PlatformIO CLI ターミナルウィンドゥ](/jp/blinky-hello-world/prerequisites.html#open-the-platformio-cli-terminal-window)で説明されているように以下のコマンドを実行します。
```bash
pio run --environment core2foraws --target monitor
```

## 検証ステップ
次の章に進む前に、デバイスが想定どおりに設定されているかを検証できます

1. AWS IoT Core コンソールのテストページを使用して、トピック `$aws/things/<<CLIENT_ID>>/shadow/update/accepted` にサブスクライブすると、すぐに **vTaskDelay()** を使用した新しいメッセージが表示されます(<<CLIENT_ID>> をデバイスのクライアント ID または画面にプリントされたシリアル番号に置き換えてください)。
2. AWS IoT Core コンソールのテストページを使用し、`$aws/things/<<CLIENT_ID>>/shadow/update` のトピックに新しいシャドウメッセージをパブリッシュします。Core for AWS IoT EduKit の LED バーが青 (COOLING) 、赤 (HEATING)、オフ (STANDBY) のパブリッシュされた値に変わります。以下はサンプルシャドウメッセージです。**hvacStatus** の値を **HEATING** または **COOLING** に設定したり、 **roomOccupied** 値を (**true** または **false** ) に切り替え、結果がどうなるかをテストします。

```
{ "state": { "desired": { "hvacStatus": "HEATING", "roomOccupancy": true } } }
```

想定どおりに機能している場合は、「[データの変換とルーティング](/jp/smart-thermostat/data-transforms-and-routing.html)」に進みましょう。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}