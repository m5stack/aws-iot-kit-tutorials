+++
title = "AWS IoT Core への接続"
weight = 30
pre = "<b>c. </b>"
+++

この章では、デバイスファームウェアの設定、構築、およびフラッシュを行います。これにより、デバイスが Wi-Fi ネットワークと AWS IoT Core に接続できるようになります。AWS IoT Core に接続して通信するには、Wi-Fi の認証情報と [AWS IoT エンドポイント](https://docs.aws.amazon.com/iot/latest/developerguide/connect-to-iot.html#iot-device-endpoint-intro) の URL を使用してデバイスを設定する必要があります。安全な MQTT 接続の確立は、[AWS IoT Device SDK for Embedded C](https://github.com/espressif/aws-iot-device-sdk-embedded-C/tree/61f25f34712b1513bf1cb94771620e9b2b001970) とオンボードの事前プロビジョニングされた安全なハードウェア証明書 [Microchip ATECC608 Trust&Go](https://www.microchip.com/wwwproducts/en/ATECC608B-TNGTLS)を利用します。セキュアエレメントを使用すると、AWS IoT Core から証明書を取得したり、独自の証明書を生成して接続したりする必要はありません。AWS IoT Device SDK for Embedded C の接続ライブラリは、AWS IoT サービスおよび機能への接続とアクセスを簡単にします。

## ESP32 ファームウェアの設定
ソースコードの設定は、[Kconfig](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html)を通して行います。Kconfig は Linux カーネルで使用されているものと同じ設定システムであり、利用可能な設定オプション (記号) をツリー構造に単純化するのに便利です。設定を行う前に、まず AWS IoT エンドポイントを取得する必要があります。

```bash
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```
AWS IoT エンドポイントを引用符なしでコピーします。`3duk1t3xampl3.iot.us-west-2.amazonaws.com` のようになります。これは、すぐ後に使用します。

設定メニューは、プロジェクトのルートで以下のコマンドを入力して、開くことができます。
```bash
pio run --environment core2foraws --target menuconfig
```
{{< img "idf_menuconfig-aws_endpoint.en.webp" "Configuring Core2 for AWS IoT EduKit with p.py menuconfig" >}}

ここでは設定を行います。キーボードの方向キーを使って、[Component config] –> [Amazon Web Services IoT Platform] に移動し、[AWS IoT Endpoint Hostname] を開いて文字列を設定します。先ほどコピーしたアドレスをボックスに貼り付けて **enter** キーを押すと、その記号が設定されます。

次に、**ESC** キーを 2 回押して設定のホーム画面に戻ります。その後、メニューから [AWS IoT EduKit Configuration] を選択します。自分の Wi-Fi 認証情報で、[WiFi SSID] と [WiFi Password] を設定します。終了したら、キーボードの **s** ボタンを押して保存し、**enter** を押してファイルの場所を確認して、**enter** を押して **q** を押して終了します。


{{% notice warning %}}
SSID が 2.4GHz ネットワーク用であることを確認してください。AWS 用 M5Stack Core2 ハードウェアの ESP32-D0WD は 5GHz 帯の Wi-Fiに対応していません。
{{% /notice %}}

## Lチカ用のファームウエアのビルド、転送、モニタリング

これまでの手順で、Lチカのファームウェアをビルド（コンパイル）してアップロードする準備ができました。この手順は「開始方法」チュートリアルと同じです。

1) ファームウエアのビルドを行うには、以下のコマンドを実行します。(コマンドの実行は数分かかります)
    ```bash
    pio run --environment core2foraws
    ```
2) ビルドが成功したら、USBで接続されたデバイスに以下のコマンドで転送します。
    ```bash
    pio run --environment core2foraws --target upload
    ```
3) 最後に、以下のコマンドでシリアルの出力を監視します:
    ```bash
    pio run --environment core2foraws --target monitor
    ```

{{% notice info %}}
シリアル出力のアップロードまたは監視中に、誤ったポートまたはタイムアウトに関するエラーが表示された場合は、`platformio.ini` ファイルを開き、そのファイルの指示に従ってアップロードポートを手動で設定してください。
{{% /notice %}}

シリアルの出力を見ていると以下のようなログが見つかります。(出力されない場合は、Core2 for AWS IoT EduKitを再起動してください)
ここに表示されているClient Idは、後で**<<CLIENT_ID>>**として利用するので、コピーしておいてください。

```
␛[0;32mI (11413) Blinky: **************************************␛[0m
␛[0;32mI (11413) Blinky: * Client Id - 123456789ABCDEF       * ␛[0m
␛[0;32mI (11423) Blinky: **************************************
```

## 章のまとめ

この章では、デバイスを正常にコンパイルしてフラッシュし、シリアル出力を監視しました。AWS IoT Device SDK for Embedded Cを使用して、[MQTT メッセージブローカー](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) (AWS IoT Core) で認証され、メッセージを受信できる状態になりました。

これで、このチュートリアルの最終章である [**Lチカ**](/jp/blinky-hello-world/blinking-the-leds.html)に進む準備が整いました。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}