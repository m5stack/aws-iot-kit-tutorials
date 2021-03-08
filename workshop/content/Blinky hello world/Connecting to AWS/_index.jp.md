+++
title = "AWS IoT Core への接続"
weight = 30
pre = "<b>c. </b>"
+++

この章では、AWS IoT Core に接続するデバイスのファームウェアの設定、ビルド、フラッシュについて説明します。AWS IoT Core に接続して通信するためには、Wi-Fi 認証情報と AWS エンドポイントの URL でデバイスを設定する必要があります。Microchip ATECC608 Trust&GO にあらかじめ証明書がプロビジョニングされているため、接続が簡素化されています。Microchip ATECC608 Trust&GO は AWS IoT のモノに割り当てられており、前章のセキュリティポリシーにアタッチされています。AWS IoT Core から証明書を取得したり、独自に生成したりする必要はありません。Espressif ESP32-D0WD でファームウェアを設定、ビルド、フラッシュするには、ESP-IDF を使用します。

## ESP32 ファームウェアの設定
ソースコードの設定は、[Kconfig](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html)を通して行います。Kconfig は Linux カーネルで使用されているものと同じ設定システムであり、利用可能な設定オプション (記号) をツリー構造に単純化するのに便利です。設定を行う前に、まず AWS IoT エンドポイントを取得する必要があります。

```bash
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```
AWS IoT エンドポイントを引用符なしでコピーします。`3duk1t3xampl3.iot.us-west-2.amazonaws.com` のようになります。これは、すぐ後に使用します。

設定メニューは、プロジェクトのルートから開くことができます。
```bash
idf.py menuconfig
```
{{< img "hello_world-menuconfig.webp" "Configuring Core2 for AWS IoT EduKit with idf.py menuconfig" >}}

ここでは設定を行います。キーボードの方向キーを使って、[Component config] –> [Amazon Web Services IoT Platform] に移動し、[AWS IoT Endpoint Hostname] を開いて文字列を設定します。先ほどコピーしたアドレスをボックスに貼り付けて **enter** キーを押すと、その記号が設定されます。

次に、**ESC** キーを 2 回押して設定のホーム画面に戻ります。その後、メニューから [AWS IoT EduKit Configuration] を選択します。自分の Wi-Fi 認証情報で、[WiFi SSID] と [WiFi Password] を設定します。終了したら、キーボードの **s** ボタンを押して保存し、**enter** を押してファイルの場所を確認して、**enter** を押して **q** を押して終了します。


{{% notice warning %}}
SSID が 2.4GHz ネットワーク用であることを確認してください。AWS 用 M5Stack Core2 ハードウェアの ESP32-D0WD は 5GHz 帯の Wi-Fiに対応していません。
{{% /notice %}}

## ESP32 ファームウェアのビルド

ファームウェアのビルドは簡単ですが、最初はかなり時間がかかります。ESP-IDF は CMAKE をビルドシステムとして使用しており、必要なファイルをすべてリンクしてから、デバイスにフラッシュ可能な ELF 形式のバイナリへのコンパイルを開始します。

```bash
idf.py build
```

## 古いファームウェアの消去
「開始方法」または 「Alexa for IoT 入門」を完了している場合、デバイスのフラッシュメモリを消去することをお勧めします。フラッシュは、アプリケーションデータ、OTA データ、キーが格納されているパーティションに分割されます。他のチュートリアルでは、このチュートリアルで使用したものとは異なるパーティションテーブルを使用しているため、消去しないと問題が発生する可能性があります。以下のコマンドの**<<DEVICE_PORT>>**をCore2 for AWS IoT EduKitが接続されているポートに書き換えて実行します。

```bash
idf.py erase_flash  -p <<DEVICE_PORT>>
```

## ESP32 ファームウェアのフラッシュとシリアルポートを介したデバイス出力のモニタリング
ビルドしたファームウェアをフラッシュし、シリアル接続でデバイスのすべての出力を表示するには、デイジーチェーンコマンドを使って次のように入力します。
```bash
idf.py flash monitor -p <<DEVICE_PORT>>
```
デバイスがフラッシュされると、再起動してアプリケーションが実行されます。デバイスが Wi-Fi ネットワークに接続し、AWS IoT Core へのセキュアな MQTT 接続を確立し、プリセットされた MQTT トピックにサブスクライブして、メッセージの送信を開始します。

## 章のまとめ
この章では、デバイスのコンパイルとフラッシュが完了し、シリアル出力をアクティブにモニタリングできるようになりました。AWS IoT Device SDK for Embedded C を使用することによって、リファレンスハードウェアは MQTT メッセージブローカー (AWS IoT Core) で認証され、メッセージを受信できる状態になりました。

これで、このチュートリアルの最終章である [**Lチカ**](/ja/blinky-hello-world/blinking-the-leds.html)に進む準備が整いました。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}