+++
title = "Espressif 証明書のフラッシュ"
weight = 20
pre = "<b>b. </b>"
+++

## 本章の概要
この章を終えるまでに、コンパニオンフォンのアプリケーションをモバイルデバイスにインストールし、必要なコードリポジトリのcloneを作成し、`Espressif Alexa の AWS アカウントを使用して` AWS IoT に接続するための証明書を受け取り、リファレンスハードウェアの分割されたフラッシュパーティションに証明書をフラッシュすることができています。

## コンパニオンモバイルアプリケーション
Alexa で認証を完了するには、デバイスを WiFi にプロビジョニングし、かつAlexa アカウントでリファレンスハードウェアをプロビジョニングするための Espressif のコンパニオンアプリが必要です。

ESP Alexa Phone Appをダウンロードする: [iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

オプションとして、[iOS](https://apps.apple.com/us/app/amazon-alexa/id944011620) と [Android](https://play.google.com/store/apps/details?id=com.amazon.dee.app) で Amazon Alexa アプリを利用できるようにすることをお勧めします (ただし、このチュートリアルでは必須ではありません)。これは、ほとんどの Alexa 対応デバイスのプロビジョニングに使用されるのと同じアプリです。

## コードへのアクセス
このチュートリアルのすべてのコードは、[Blinky Hello World(Lチカ)](/ja/blinky-hello-world.html) チュートリアルでcloneを作成したリポジトリの `Alexa_for_IoT-Intro` フォルダにあります。リポジトリのcloneを再度作成するには下記を実行してください:
```
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```

## AWS IoT 証明書を設定する
AWS IoT コアと通信するために、AWS IoT 証明書を作成する必要があります。Espressif は、このワークショップとデバイスのために 、AWS IoT EduKit リファレンスハードウェア用のM5Stack Core2 で使用できる AWS IoT 証明書を提供しています。

[こちら](https://espressif.github.io/esp-va-sdk/#/) の手順に従って、証明書を取得します。

ダウンロードした証明書を解凍すると、**espcredentials** というフォルダが作成されます。フォルダに移動し、**mfg_config.csv** (証明書に付属) を修正し、各ファイルに正しいパスを反映します。次に、ターミナルでこのフォルダに移動し、下記のコマンドを実行し、証明書をデバイスにフラッシュします。**<<DEVICE_PORT>>** をデバイスポートに置き換えます。デバイスポートがわからない場合は、「Blinky Hello World(Lチカ)」 の例にある[ホストマシンのシリアルポート識別](/ja/blinky-hello-world/device-provisioning.html#heading)のインストラクションに従ってください。

```bash
python $IDF_PATH/components/nvs_flash/nvs_partition_generator/nvs_partition_gen.py generate /path/to/mfg_config.csv mfg.bin 0x6000

python $IDF_PATH/components/esptool_py/esptool/esptool.py --chip esp32 --port <<DEVICE_PORT>> write_flash 0x10000 mfg.bin
```

設定がすべて完了し準備が整ったら、[AFI のビルドとテスト](/ja/intro-to-alexa-for-iot/building-and-testing-afi.html)に進みましょう。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}