+++
title = "AFI のビルドとテスト"
weight = 30
pre = "<b>c. </b>"
+++

## 本章の概要
この章では、移植された ESP VA-SDK ファームウェアをビルドし、それをデバイスにフラッシュし、Wi-Fi をプロビジョニングしてデバイスを Alexa アカウントに認証させて、AFI のこのベータ版で利用可能な Alexa 音声コマンドを使用していくつかのスマートホーム機能をテストします。

## ファームウェアをフラッシュする
PlatformIO CLI を使用して、ファームウェアをコンパイルし、ファームウェアをアップロードし、デバイスのシリアル出力を監視します。アプリのビルドとフラッシュには時間がかかりますが、それが完了すると、端末にデバイスログのストリームが表示されるはずです。ポートが自動検出されないというエラーが表示される場合は、[USBポートの確認](/jp/getting-started/prerequisites.html) の説明を参照して、コマンドを再試行してください。**Ctrl** + **C** キーストロークの組み合わせでモニターセッションを閉じることができます。
   ```bash
   pio run --environment core2foraws --target upload --target monitor 
   ```

## デバイスをプロビジョニングする
プロビジョニングを処理するために、Wi-Fi ネットワーク認証情報を構成し、ESP Alexa Phone Application を使用して Alexa アカウントでアプリケーションを認証する必要があります。

モバイルアプリストアからアプリケーションをダウンロードします。[iOS](https://apps.apple.com/in/app/esp-alexa/id1464127534) / [Android](https://play.google.com/store/apps/details?id=com.espressif.provbleavs)

プロビジョニングの手順
1. コンパニオンアプリを起動します。
  - Bluetooth が有効で、アプリには Bluetooth にアクセスするための適切な権限があることを確認してください。
2. オプションの [Add New Device (新しいデバイスを追加)] を選択します。
3. お使いのデバイスがアプリに表示されます。
  - `abcdf1234` などの値が表示された**Proof of Possession** モーダルウィンドウが開きます。完了を押して続行します。
4. デバイスに接続した後、amazonのアカウント(ECサイト向けの)にサインインします。
5. Wi-Fi ネットワークを選択し、認証情報を入力します。
6. Wi-Fi 接続が成功すると、サンプル発話のリストが表示されます。

## Alexa の使用
上記の手順が完了すると、シリアルモニターに以下のようないくつかのログが表示されます。

```bash
I (17325) [http_transport]: Subscribing /capabilities/acknowledge...
I (17535) [http_transport]: Subscribing /connection/fromservice...
I (17735) [http_transport]: Subscribing /directive...
I (17945) [http_transport]: Subscribing /speaker...
...
I (20735) [directive_proc]: Name: EndpointForwarding
...
I (22675) [directive_proc]: Name: SetAttentionState
...
E (22685) [app_va_cb]: Enabling Mic
```

Alexa と対話するには、デバイスに Alexa と話しかける必要があります。それによって、デバイスで実行されている**Espressif Wake Word Engine** がトリガーされ、**LISTENING** アテンション状態に入ります。さまざまなアテンション状態の詳細については、[ドキュメンテーション](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/ux-design-attention.html#states)を参照してください。オーディオキャプチャの詳細については、[SpeechRecognizer API](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/avs-speechrecognizer-concepts.html) ドキュメンテーションを参照してください。

{{< img "speechrecognizer-state.en.png" "Audio Capture Speech Recognizer Attention States" >}}

{{% notice info %}}
他の Alexa デバイスと同様に、デバイスが IDLE 状態のときは、キーワード「Alexa」だけをリッスンします。ひとたびキーワードがトリガーされると、デバイスはクラウドへのオーディオストリーミングを開始します。
{{% /notice %}}

Alexa にさまざまな言葉をかけてみてください。**Alexa** が聞き取ると側面の LED が青く点灯します (Alexa が「スリープ解除」しない場合は、デバイスに近づいて話しかけてみてください):

{{% notice info %}}
**Espressif Alexa の AWS アカウント** を利用してセットアップする場合は、英語が初期設定されています。日本語を試したい場合は、ンパニオンアプリからデバイスを選択し、言語を日本語に切り替えてから試してください。
{{% /notice %}}


* _Alexa, what time is it?_
* _Alexa, tell me a joke?_
* _Alexa, turn on all of the lights_ 
  * 同じアカウントですでにいくつかの Alexa のスマートホームデバイスを使用している場合にのみ機能します

{{< img "alexa-time.en.webp" "Alexa, what time is it?">}}

## Alexa スマートホーム機能 (ベータ版) のテスト
AFI デバイスには **Alexa Built-In** が搭載されています。これは Alexa に話しかけるには直接デバイスに話しかければよく、Alexa はデバイスから音声で応答することを意味します。しかし、Espressif の AFI のこのバージョンはベータ機能として Alexa スマートホームコマンドもサポートしているため、デバイスの属性を制御することができます。

Alexa for AWS IoT サンプルアプリケーションは Alexa アプリ内に **Light** という仮想デバイスを作成し、そうすることで 2 つのインターフェイスをサポートします。

* [PowerController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-powercontroller.html) はライトの点灯・消灯を行います。
* [RangeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-rangecontroller.html) はデバイスの明るさを調整します。

![The device named "Light" should show up in your Alexa App](building-and-testing-afi/alexa_app-light_Device.en.png?height=500px)

仮想デバイスであるため、更新されたステータスを画面に出力します。これは、音声または Alexa アプリを介してテストできます。

* 音声による場合 - `アレクサ、ライトをつけて` と言うと、正常であれば、Alexa は「OK」またはその他の確認音で応答します。
* Alexa アプリ経由の場合 - Alexa スマートフォンアプリ (ESP Alexa スマートフォンアプリではありません) を開き、[デバイス] に進み、次に**Lights** または**All Devices**のどちらかに進むと、**Light**というデバイスが表示されます (下のスクリーンショットを参照してください)。電源アイコンをタップすると、アイコンのオフとオンが切り替わることがわかります。

いずれの方法でも、端末に以下のようなメッセージが表示されます。
```bash
I (97445) [alexa_smart_home]: Namespace: Alexa.PowerController, Name: TurnOn
```

同じように、次のいずれかの方法で明るさの範囲を調整できます。

* 音声による場合 - Alexa、_Alexa, set brightness on the light to 80_ と言うと、正常であれば、Alexa は「OK」またはその他の確認音で応答します。
* Alexa アプリ経由の場合 - 明るさのスライダーで調整します。

いずれの方法でも、シリアルモニターに以下のようなメッセージが表示されるます。
```bash
[app_smart_home]: *************** Light's Brightness changed to 60 ***************
```

これは、Alexa を搭載したデバイスには音声アシスタントがあるということのみならず、Alexa を使ってデバイス自体のプロパティを制御できるという点でも、便利です!

次は [カスタムスマートホームデバイス](/jp/intro-to-alexa-for-iot/custom-smart-home-device.html)の作成です。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}