+++
title = "ESP RainMaker Agent の実行"
weight = 12
pre = "<b>b. </b>"
+++

アプリケーションをコンパイルしてデバイスにフラッシュし、ESP Rainmaker スマートフォンアプリを介して Wi-Fi をプロビジョニングし、デバイスを割り当てるユーザーを登録して、AWS IoT を介して周辺機器を制御する準備が整いました。

## PlatformIO でプロジェクトを開く
cloneしたプロジェクトやダウンロードして抽出したプロジェクトのルートには、いくつかのフォルダがあります。このチュートリアルでは、PlatformIO で Getting-Started プロジェクトを開きます。まず Visual Studio Code を開き、PlatformIO 拡張機能がロードされるまで数秒待った後、左バーの PlatformIO ロゴをクリックします。[Open Project (プロジェクトを開く)] を選択し、`Core2-for-AWS-IoT-EduKit/Getting-Started` フォルダに移動し、[Open] をクリックします。
{{< img "pio_home.png" "PlatformIO home screen" >}}

## ホストマシンのシリアルポートの特定と設定

ファームウェアをアップロードしたり、シリアル出力をモニタリングしたりするためには、デバイスとの通信に使用するポートを把握しておく必要があります。macOSの場合、デバイスは通常 `/dev/cu.SLAB_USBtoUART` にあり、すでにある設定を変更する必要はありません。Linuxをお使いの場合は、通常`/dev/ttyUSB0`にあり、dialoutグループにユーザーを追加する必要があります。 Windowsをご利用の場合は、`COM`に続く数字を修正する必要があります(例えば、COM3)。追加の設定が必要な Windows や一部の Linux ディストリビューションの場合は、ESP32 とのシリアル接続を確立するための方法を Espressif の[公式ドキュメント](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/establish-serial-connection.html)でご確認ください。

Visual Studio Code ファイルエクスプローラー (左バーの<i class="far fa-copy"></i>) を選択し、**platformio.ini** ファイルを開き、**upload_port** の値をデバイスが接続されているシリアルポートに合わせて変更します。(`;`でコメントアウトされていれば、`;`を削除します)

{{% notice note %}}
[upload_port](https://docs.platformio.org/en/latest/projectconf/section_env_upload.html#upload-port)オプションについては、PlatformIO の公式ドキュメントでフォーマットやその他のサンプルをご確認ください。
{{% /notice %}}

## RainMaker Agent ファームウェアのビルドとアップロード
これでファームウェアをビルドしてフラッシュする準備ができました。左バーの PlatformIO ロゴをクリックして開始します。次に、<i class="fas fa-redo"></i> アイコンをクリックして (**PROJECT TASK**の横にマウスを置いて)、利用可能なオプションを更新します。[Build] ボタンをクリックして、デバイスのファームウェアのビルドを開始します。コンピュータにもよりますが、数分かかります。これが完了したら(SUCCESSと表示されたら)、PlatformIO プロジェクトのタスクメニューから [Upload and Monitor (アップロードとモニタリング)] オプションをクリックします。アップロードが正常に完了すると、コンパイルしてアップロードしたばかりのファームウェアでデバイスが起動します。また、その端末のビューポート (現在のシリアルモニターウィンドウ) に、デバイスからのシリアル出力が表示されるようになります。デバイスは、セキュリティキーを生成するプロセスを経て、自己登録を実行します。キーの生成には数秒から数分かかりますが、登録が完了するとシリアルモニターのウィンドウに QR コードが表示されます。
{{< img "pio_menu.png" "PlatformIO Menu to build, flash, monitor" >}}

{{% notice note %}}
その他のトラブルシューティングや FAQ については、[Espressif RainMakerの公式FAQ](https://rainmaker.espressif.com/docs/faqs.html)を参照してください。
{{% /notice %}}

## デバイスの登録とプロビジョニング
スマートフォンで ESP RainMaker スマートフォンアプリを開き、必要なモバイルアプリの許可を付与し、[デバイスの追加] を押してから、パソコンのシリアルモニターに表示される QR コードをスキャンします。その後、2.4GHz ワイヤレスネットワークの Wi-Fi 認証情報を使用した Wi-Fi プロビジョニングなどのプロビジョニングプロセスが実行されます。Wi-Fi 接続が成功すると、デバイスが自身の認証を行い、閲覧または制御可能な複数の仮想デバイスがスマートフォンアプリに表示されます。1～2 分経ってもスマートフォンアプリで仮想デバイスが**オフライン**と表示されている場合は、下にスクロールしてリフレッシュしてみてください。
{{< img "qr_code_scan.png" "Scan the QR code in serial output" >}}

仮想デバイスが一覧表示され、モバイルアプリでオンラインになったら、オンボードモーターや LED のオン/オフ、モーターの速度調整、LED バーの色や明るさの設定、デバイス内部の温度の確認などを行うことができます。

{{% notice info %}}
間違った Wi-Fi 認証情報を入力した場合は、まず[ファームウェアを削除](/ja/getting-started/run-rainmaker.html#platformio--1)し、Espressif RainMaker Agent ファームウェアをデバイスに再アップロードし、スマートフォンにデバイスを追加し直す必要があります。
{{% /notice %}}

## PlatformIO でファームウェアの削除
このアプリケーションの作業が完了し、他のチュートリアルに移動する準備ができたら、デバイスのファームウェアを消去する必要があります。
ただし、ポートへの他の通信をブロックしているため、最初にアクティブなシリアルモニターを停止する必要があります。シリアルモニターを停止するには、Visual Studio Codeのシリアルモニタが表示されているペインを選択し、**CTRL** + **C** で停止します。次に、PlatformIO メニューを開いていない場合はメニューを開き、[プラットフォーム] メニューを展開して、[Erase Flash (フラッシュを削除)] オプションをクリックします。これにより、使用された RainMaker 証明書を含む、デバイス上のすべてのデータが削除されます。
{{< img "pio_platform_menu.png" "Erasing flash with PlatformIO" >}}

{{% notice note %}}
フラッシュを削除すると、デバイスの画面は何も表示されず、スピーカーから音が出ます。これはアプリケーションが何もない状態の正常な動きです。
{{% /notice %}}

## まとめ
これで、AWS IoT EduKit プログラムを利用してコネクテッドホームアプリケーションを構築することができました。 それだけでなく、デバイスに埋め込まれたコードを作成、編集、コンパイル、フラッシュするために必要なツールも手に入りました。 次のチュートリアルはより実践的な内容になり、独自のエンドツーエンド IoT ソリューションの構築を開始するためのスキルを学びます。

次の章 [Blinky Hello World(Lチカ)](/ja/blinky-hello-world.html)にお進みください。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}