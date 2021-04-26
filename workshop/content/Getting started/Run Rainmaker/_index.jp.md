+++
title = "ESP RainMaker Agent の実行"
weight = 12
pre = "<b>b. </b>"
+++

アプリケーションをコンパイルしてデバイスにフラッシュし、ESP RainMaker スマートフォンアプリを介して Wi-Fi をプロビジョニングし、デバイスを割り当てるユーザーを登録して、AWS IoT を介してバーチャルスマートホームデバイスを制御する準備が整いました。

## PlatformIO でプロジェクトを開く
cloneしたプロジェクトやダウンロードして抽出したプロジェクトのルートには、いくつかのフォルダがあります。このチュートリアルでは、PlatformIO で Getting-Started プロジェクトを開きます。まず Visual Studio Code を開き、PlatformIO 拡張機能がロードされるまで数秒待った後、左バーの **PlatformIO ロゴ** をクリックします。 **[Open Project (プロジェクトを開く)]** を選択し、`Core2-for-AWS-IoT-EduKit/Getting-Started` フォルダに移動し、**Open** をクリックします。
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - PIOメニューを開く。 2 - PIO Homeを開く。 3 - Getting Startedプロジェクトを開く" >}}

## RainMaker Agent ファームウェアのビルドとアップロード
これで、RainMaker Agent ファームウェアをビルド（コンパイル）してアップロードする準備ができました。コンパイルは [GCC コンパイラ](https://gcc.gnu.org/onlinedocs/gcc/) によって行われ、人間が読めるコードを [オブジェクトコード](https://en.wikipedia.org/wiki/Object_code) ([elf](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)) とバイナリファイル (ファームウェア) に変換します。これらのファイルは、デバイスのオンボードフラッシュにアップロードされ、仮想シリアルポートを介して実行されます。シリアルポート(UART 経由) は双方向通信が可能なため、デバイスからホストマシンにデータを受信できます。ファームウェアをビルドしてアップロードし、PlatformIO を使用してシリアルポートを介してデバイスからの出力を監視するには以下の手順を実行します。

1) VS Codeのアクティビティーバー(左のメニュー)で **PlatformIOのロゴ** をクリックします
2) **Quick Access** メニューの、**Miscellaneous** にある **New Terminal** をクリックしてターミナルを開きます
3) 開かれたターミナルの中で、以下のコマンドを実行してファームウエアのビルドをおこないます。(コマンドの実行は数分かかります)
    ```bash
    pio run --environment core2foraws
    ```
4) ビルドが成功したら、同じターミナル上から以下のコマンドを実行して、USBで接続されたデバイスに以下のコマンドで転送します。
    ```bash
    pio run --environment core2foraws --target upload
    ```
5) 最後に、同じターミナルで以下のコマンドでシリアルの出力を監視します。
    ```bash
    pio run --environment core2foraws --target monitor
    ```

{{% notice info %}}
シリアル出力のアップロードまたは監視中に、誤ったポートまたはタイムアウトに関するエラーが表示された場合は、`platformio.ini` ファイルを開き、そのファイルの指示に従ってアップロードポートを手動で設定してください。
{{% /notice %}}

## デバイスの登録とプロビジョニング
アップロードが正常に完了すると、コンパイルおよびアップロードされたばかりのファームウェアでデバイスが起動します。また、ターミナル上にデバイスからのシリアル出力の表示が開始されます。デバイスは、セキュリティキーを生成し、[Assisted Claiming (ESP32)](https://rainmaker.espressif.com/docs/claiming.html#assisted-claiming-esp32) を実行するプロセスを実行します。キーの生成には数秒から数分かかりますが、クレームが完了すると QR コードがターミナルビューポートに表示されます。(QRコードが表示されない場合はケーブルを繋いだまま、Core2 for AWS IoT EduKitを再起動してください)

スマートフォンで ESP RainMaker スマートフォンアプリを開き、必要なモバイルアプリの許可を付与し、 **デバイスの追加** を押してから、パソコンのシリアルモニターに表示される QR コードをスキャンします。その後、2.4GHz ワイヤレスネットワークの Wi-Fi 認証情報を使用した Wi-Fi プロビジョニングなどのプロビジョニングプロセスが実行されます。Wi-Fi 接続が成功すると、デバイスが自身の認証を行い、閲覧または制御可能な複数の仮想デバイスがスマートフォンアプリに表示されます。1～2 分経ってもスマートフォンアプリで仮想デバイスが**オフライン**と表示されている場合は、下にスクロールしてリフレッシュしてみてください。
{{< img "pio-qr_code_scan.en.png" "Scan the QR code in serial output" >}}

仮想デバイスが一覧表示され、モバイルアプリでオンラインになったら、オンボードモーターや LED のオン/オフ、モーターの速度調整、LED バーの色や明るさの設定、デバイス内部の温度の確認などを行うことができます。

{{% notice info %}}
間違った Wi-Fi 認証情報を入力した場合は、まず[ファームウェアを削除](/jp/getting-started/run-rainmaker.html#platformio--1)し、Espressif RainMaker Agent ファームウェアをデバイスに再アップロードし、スマートフォンにデバイスを追加し直す必要があります。
その他のトラブルシューティングやFAQについては、公式の [Espressif RainMaker FAQs](https://rainmaker.espressif.com/docs/faqs.html)を参照してください。
{{% /notice %}}

## PlatformIO でファームウェアの削除
このアプリケーションの作業が完了し、他のチュートリアルに移動する準備ができたら、デバイスのファームウェアを消去する必要があります。
ただし、ポートへの他の通信をブロックしているため、最初にアクティブなシリアルモニターを停止する必要があります。シリアルモニターを停止するには、Visual Studio Codeのシリアルモニタが表示されているペインを選択し、**CTRL** + **C** で停止します。ファームウエアを削除するには以下のコマンドを実行します。
```bash
pio run --environment core2foraws --target erase
```


{{% notice note %}}
ファームウエアを削除すると、デバイスの画面は何も表示されず、スピーカーから音が出ます。これはファームウエアが何もない状態の正常な動きです。
{{% /notice %}}

## まとめ
これで、AWS IoT EduKit プログラムを利用してコネクテッドホームアプリケーションを構築することができました。 それだけでなく、デバイスに埋め込まれたコードを作成、編集、コンパイル、フラッシュするために必要なツールも手に入りました。 次のチュートリアルはより実践的な内容になり、独自のエンドツーエンド IoT ソリューションの構築を開始するためのスキルを学びます。

次の章 [Lチカ](/jp/blinky-hello-world.html)にお進みください。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}