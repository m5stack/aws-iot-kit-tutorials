+++
title = "Espressif 証明書のフラッシュ"
weight = 20
pre = "<b>b. </b>"
+++

## 本章の概要
この章を終えるまでに、コンパニオンフォンのアプリケーションをモバイルデバイスにインストールし、必要なコードリポジトリのcloneを作成し、`Espressif の Alexa で利用している AWS アカウントを使用して` AWS IoT に接続するための証明書を受け取り、リファレンスハードウェアの分割されたフラッシュパーティションに証明書をフラッシュすることができています。

## コンパニオンモバイルアプリケーション
Alexa で認証を完了するには、デバイスを Wi-Fi にプロビジョニングし、かつAlexa アカウントでリファレンスハードウェアをプロビジョニングするための Espressif のコンパニオンアプリが必要です。

ESP Alexa Phone Appをダウンロードする: [iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

オプションとして、[iOS](https://apps.apple.com/us/app/amazon-alexa/id944011620) と [Android](https://play.google.com/store/apps/details?id=com.amazon.dee.app) で Amazon Alexa アプリを利用できるようにすることをお勧めします (ただし、このチュートリアルでは必須ではありません)。これは、ほとんどの Alexa 対応デバイスのプロビジョニングに使用されるのと同じアプリです。

## コードへのアクセス
このチュートリアルのすべてのコードは、[Lチカ](/jp/blinky-hello-world.html) チュートリアルでcloneを作成したリポジトリの `Alexa_for_IoT-Intro` フォルダにあります。再度Cloneする場合は、[PlatformIO CLI ターミナルウィンドゥ](/jp/blinky-hello-world/prerequisites.html#platformio)で以下のコマンドを実行します。
```
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```

## プロジェクトを開く
このチュートリアルでは、Alexa_for_IOT-Intro プロジェクトを使用します。新しいVS Codeウィンドウで、 
1. VS Codeアクティビティバー（左端のメニュー）の **PlatformIOのロゴ** をクリックします
2. 左の PlatformIO メニューから **Open** を選択します。
3. **Open Project** をクリックします
4. `Core2-for-AWS-IOT-Edukit/Alexa_for_IOT-intro` フォルダに移動し、**Open** をクリックします。
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - PIOメニューを開く。 2 - PIO Homeを開く。 3 - プロジェクトフォルダを開く" >}}

次に、新しいPlatformIO CLIターミナルウィンドウをVS Codeで開く必要があります。
1. VS Codeのアクティビティーバー(左のメニュー)で **PlatformIOのロゴ** をクリックします
2. **Quick Access** メニューの、**Miscellaneous** にある **New Terminal** をクリックしてターミナルを開きます

{{< img "pio-new_terminal-smart_thermostat.en.png" "PlatformIO CLI端末をVS Codeで表示" "1 - PIOメニューを開く。2 - 新しい PIO ターミナルを開く。3 - 'PlatformIO CLI'ターミナルセッションにいることを確認する。">}}

## AWS IoT 証明書を設定する
AWS IoT コアと通信するために、AWS IoT 証明書を作成する必要があります。Espressif は、このワークショップとデバイスのために 、AWS IoT EduKit リファレンスハードウェア用のM5Stack Core2 で使用できる AWS IoT 証明書を提供しています。

[こちら](https://espressif.github.io/esp-va-sdk/) の手順に従って、証明書を取得します。

証明書のzip ファイルを含む電子メールを受信したら、ファイルを保存し、内容を解凍する必要があります。解凍すると、 **espcredentials** というフォルダが作成されます。デバイスを接続したら、PlatformIO CLI ターミナルウィンドウで次のコマンドを入力して、これらの証明書をデバイスにアップロードできます。

{{%expand "Ubuntu または macOS" %}}
1. espcredentialsの内容を `Core2-for-AWS-IoT-EduKit/Alexa_for_IoT-Intro/esp_alexa_credentials` にコピーします(ただし、**mfg_config.csv** を除く)。  [rsync](https://download.samba.org/pub/rsync/rsync.1)のコマンドを利用してコピーをしますが、 **<<PATH_TO>>** を **espcredentials** フォルダのパスに修正して実行してください。
   ```bash
   rsync -avr --exclude='mfg_config.csv' <<PATH_TO>>/espcredentials/ ./esp_alexa_credentials/
   ```
   
2. 以下の PIO コマンドを使用してスクリプトを実行します。このコマンドは、1) esp_alexa_credentials フォルダーに `mfg.bin` という認証情報ファイルを含むバイナリファイルを作成し、そのバイナリをオンボードの [不揮発性フラッシュストレージ](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/storage/nvs_flash.html) (NVS) のパーティションにアップロードします。これらはアプリケーションから後で参照されます。
   ```bash
   pio run --environment core2foraws --target flash_certificates
   ```
{{% /expand%}}
{{%expand "Windows" %}}
1. espcredentialsの内容を `Core2-for-AWS-IoT-EduKit/Alexa_for_IoT-Intro/esp_alexa_credentials` にコピーします(ただし、**mfg_config.csv** を除く)。  [rsync](https://download.samba.org/pub/rsync/rsync.1)のコマンドを利用してコピーをしますが、 **<<PATH_TO>>** を **espcredentials** フォルダのパスに修正して実行してください。
   ```PowerShell
   robocopy "<<PATH_TO>>\espcredentials\" ".\esp_alexa_credentials\" /xf mfg_config.csv
   ```
   
2. 以下の PIO コマンドを使用してスクリプトを実行します。このコマンドは、1) esp_alexa_credentials フォルダーに `mfg.bin` という認証情報ファイルを含むバイナリファイルを作成し、そのバイナリをオンボードの [不揮発性フラッシュストレージ](https://docs.espressif.com/projects/esp-idf/en/v4.2/esp32/api-reference/storage/nvs_flash.html) (NVS) のパーティションにアップロードします。これらはアプリケーションから後で参照されます。
   ```PowerShell
   pio run --environment core2foraws --target flash_certificates
   ```
{{% /expand%}}

設定がすべて完了し準備が整ったら、[AFI のビルドとテスト](/jp/intro-to-alexa-for-iot/building-and-testing-afi.html)に進みましょう。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}