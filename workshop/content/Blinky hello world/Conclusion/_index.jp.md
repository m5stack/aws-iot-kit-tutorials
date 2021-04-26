+++
title = "まとめ"
weight = 50
pre = "<b>e. </b>"
+++

この章を通して、クラウドに繋がったデバイスでのLチカを体験することができたと思います。また、PlatformIO を使用すると、AWS IoT EduKit のファームウェアを Core2 を設定、ビルド、フラッシュ、モニタリングすることができました。これで、デバイスが AWS IoT Thing として AWS アカウントに登録され、オンボードのセキュアチップを使用したセキュアなクラウド接続が可能になります。AWS IoT Core に接続し、MQTT 経由でデバイスからメッセージを送信し、AWS IoT コンソールの MQTT クライアントから MQTT メッセージを受信しました。このメッセージは、デバイスのLEDを切り替えるために使用されました。

このAWS IoT EduKit プログラムでは C言語について解説したり、プログラミングすることを要求することは有りませんが、実際の組み込み開発には不可欠となります。C言語を理解することで、ハードウェアを最大限に活用できます。もし、C言語を使ったプログラミングに慣れていないが学ぶことに興味がある、またはしばらく使っていないが、AWS IoT EduKitプログラムで提供するアプリケーションコードをもっと理解したい場合、Jens Gustedtはクリエイティブ・コモンズ・ライセンスの下で [Modern C](https://modernc.gforge.inria.fr/) を提供しており、こちらから[ダウンロード](https://modernc.gforge.inria.fr/download.html)することができますので参考にしてみてください。

## デバイスで何がおきていたか？

## FreeRTOSのカーネル

デバイス上のアプリケーションは、[FreeRTOS カーネル](https://www.freertos.org/)を使用しています。リアルタイムオペレーティングシステム(RTOS)について詳しくは[こちら](https://www.freertos.org/about-RTOS.html)を参照してください。これは、コンパイルしたバイナリ (および他のライブラリやドライバー) にアプリケーションコードとともにバンドルされます。FreeRTOS カーネルは、シーケンシャルコードのブロック ([タスク](https://www.freertos.org/taskandcr.html)と呼ばれる) を、プロセッサで実行する時間を持つようにスケジュールします。この章のLチカではAWS に送信するメッセージの作成、受信したメッセージの処理、表示の更新、LED の点滅など、永続的にループするタスクがあります。 

AWS IoT EduKit 用 Core2 は 240 MHz デュアルコアプロセッサをパックしていますが、一般的なPCのような 4 GHz以上や、 8コア以上と比較すると、非常に性能が限られています。このようなデバイスの貴重なプロセッサ時間を最大限に活用するために、FreeRTOS カーネルは [タスク状態](https://www.freertos.org/RTOS-task-states.html) を実行中や中断などの [状態] にします。FreeRTOSカーネルは、マイクロコントローラアプリケーションに、別のタスクが中断またはブロックされた状態になった後、実行する個々のタスクを迅速に切り替えることで、プロセッサを最適化する機能を提供します (下図参照)。これにより、同時実行性が向上し、単純な [Super Loop Architecture](https://en.wikibooks.org/wiki/Embedded_Systems/Super_Loop_Architecture) と比較して、組み込みアプリケーションのパフォーマンスが大幅に向上します。
{{< img "RTOS_scheduling_execution.en.png" "Example of FreeRTOS application scheduling" "1 - MQTTの開始。 2 - MQTT タスクのブロック & 画面表示タスク開始, 3 - 画面表示タスクのブロック & MQTTタスクの実行、メッセージング。 4 - MQTTタスクのブロック & LED点滅タスクの実行(LED on)。 5 - 点滅タスクのブロック & 画面表示タスクの実行, 6 - 画面表示タスクのブロック & 点滅タスクの実行(LED off), 7 - 点滅タスクのブロック & 画面表示タスクの実行">}}

Super Loop Architectureのプログラミングは簡単ですが、コードの実行をブロック/遅延するコード領域は、MCUがアイドル状態であることを意味します。これにより、さまざまなレイテンシーと帯域幅によるクラウド接続、コードのブロックによるデータの損失やアクションの実行が重要であるアプリケーション、または単にスムーズなディスプレイ画面を動かすために、RTOS が不可欠になります。
{{< img "super_loop_execution.en.png" "Example of super loop application execution" "1 - MQTTの処理を中断, 2 - MQTTメッセージの処理中断 & LEDの点灯、,500msスリープ、 LEDの消灯、 500msスリープ。 3 - 点滅処理の終了 & ディスプレイの再表示。 4 - ディスプレイの操作終了 & LEDの点灯、,500msスリープ、 LEDの消灯、 500msスリープ" >}}

もっと詳しく、RTOSについて学びたい場合は、この[ビデオシリーズ](https://www.youtube.com/watch?v=F321087yYy4)を参照してください。FreeRTOSについて学びたい場合は、[公式のサイト](https://www.freertos.org/RTOS.html)や、[AWS Black Belt Online Seminar](https://aws.amazon.com/jp/blogs/news/webinar-bb-freertos-2020/)を御覧ください。

## AWS IoT Device SDK for Embedded C

この章で使用された [AWS IoT Device SDK for Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) では、AWS IoT との接続とメッセージングが簡素化され、付属のセキュアエレメントライブラリと連携してデバイスを作成することができました認証は簡単です。組み込みC用のデバイスSDKは、`Core2-for-AWS-IoT-EduKit/Blink-Hello-World/components/esp-aws-iot/` フォルダにあります。

## The Board Support Package — Device Drivers

M5Stack Core2 for AWS IoT EduKit リファレンスハードウェアには、スクリーン、パワーマネージメントチップ、セキュアエレメント、スピーカー、マイク、6軸 [慣性測定ユニット](https://en.wikipedia.org/wiki/Inertial_measurement_unit) (IMU)、タッチドライバ、LEDバーの様な機能があり、board support package (BSP)として提供されています。BSP は、ハードウェアの機能にアクセスするための簡単な方法を提供する API を備えたドライバーです。BSPは、`Core2-for-AWS-IOT-Edukit/blink-Hello-world/Components/core2foraws/` フォルダにあります。この章では、SK6812ドライバを使用してサイドLEDバーを制御し、[LVGL ライブラリ](https://docs.lvgl.io/v7/en/html/) を使用してディスプレイ上に表示しました。

次の章は、[**スマートサーモスタット**](/jp/smart-thermostat.html)です。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}