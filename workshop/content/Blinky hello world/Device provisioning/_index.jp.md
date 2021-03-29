+++
title = "デバイスのプロビジョニング"
weight = 20
pre = "<b>b. </b>"
+++

この章では、オンボードの Microchip ATTECC608 Trust&GO セキュアエレメントを使用して AWS IoT Core に接続するためのデバイスをプロビジョニングし、TLS 接続を確立します。組み込みのハードウェアの信頼の基点により、プライベートキーを公開することなく、簡素化された迅速なプロビジョニングパスを持つことができます。デバイスに組み込まれているデバイス証明書を取得し、マニフェストファイルを作成して AWS IoT のモノ (デバイスの表現と記録) を作成することができます。このデバイスのクライアント ID は、AWS IoT Core に登録され、シリアル番号で識別されます。同様のプロセスを使用することによって、一度に何千、何百万ものデバイス群のデプロイを自動化することができます。

## ホストマシンのシリアルポートの特定
ESP32 とのシリアル接続の確立については、Espressif の[公式ドキュメント](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/establish-serial-connection.html)を参照してください。お使いの端末のポートは、お使いの OS によって異なります。macOSの場合、デバイスは通常 `/dev/cu.SLAB_USBtoUART` にあります。Linuxをお使いの場合は、通常`/dev/ttyUSB0`にあります。 Windows の場合、通常は `COM` で始まり、末尾が数字になります。


## デバイス証明書の取得とAWS IoTへthingの登録
ここではCore2 for AWS IoT EduKitリファレンスハードウェアのセキュアエレメントからデバイス証明書を取得し、x.509証明書（AWS IoTのregistration codeを含む）でデバイス証明書に署名してデバイスマニフェストを生成し、AWS IoTにデバイス証明書を作成し、IoT Policyを登録するプロセスを簡素化しています。

`Espressif Crypto Auth Utility` ディレクトリに移動し、pipコマンドで必要な依存関係をインストールします。
```bash
cd Core2-for-AWS-IoT-EduKit/Blinky-Hello-World/utilities/AWS_IoT_registration_helper/
CRYPTOAUTHLIB_NOUSB=1 python3 -m pip install --no-cache -r requirements.txt
```

次に、必要なすべての手順を実行してくれるPythonスクリプトを実行します。引数の**<<DEVICE_PORT>>**には、Core2 for AWS IoT EduKitが接続されているポートを指定します。
```bash
python registration_helper.py -p <<DEVICE_PORT>>
```

{{% notice info %}}
シェルを閉じたり、新しいシェルを開いたりする場合は、毎回 `conda activate edukit` を再入力して、環境を再起動し、ESP-IDF toolをパスに追加するために、`export.sh` (macOS/Linux) or `export.bat` (Windows) を実行します。
{{% /notice %}}

デバイスが正しくAWS IoTに登録されたら、以下のコマンドを入力して**Blinky-Hello-World**ディレクトリに移動します。
```bash
cd ../..
```

## 章のまとめ
この章では、セキュアエレメントを使用して AWS IoT の[モノ](https://docs.aws.amazon.com/iot/latest/developerguide/thing-registry.html)を作成し、モノ用の[IoTポリシー](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html)を設定し、モノに[デバイス証明書](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html)をアタッチしました。セキュアエレメントを利用することで、プライベートキーを公開したりすることなく安全にプロビジョニングが実行されます。

「[AWS IoT Core への接続](/ja/blinky-hello-world/connecting-to-aws.html)」に進みます。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}