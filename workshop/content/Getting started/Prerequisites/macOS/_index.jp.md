+++
pre = "› "
linkTitle = "macOS"
title = "MacOS 10.14+ のセットアップ"
+++

以下に、macOS 環境でセットアップして GitHub リポジトリからコードをダウンロードし、コードを表示および編集し、ハードウェアで使用できるようにコードをコンパイルし、ハードウェアのフラッシュメモリにコードをアップロードする手順を示します。これらのインストール手順は、RainMaker プラットフォーム用の Espressif の AWS アカウントとサービスを使用する **開始方法** チュートリアルで必要となります。

## Silicon Labs USB to UART bridgeのセットアップ
AWS IoT EDUKit for Core2 は、Silicon Labsの CP210x USB から UART ブリッジを介してホストマシンと通信します。オンボードCP2104は、ESP32-D0WDマイクロコントローラとのホスト通信を容易にするUSB-to-UARTブリッジです。マイクロコントローラは、[UART](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/uart.html)0を介して双方向に通信します。CP210xは、USB-Cを介して確立されたホストマシン上の仮想通信ポートを介して変換します。仮想シリアルポートとその間の通信のマウントを有効にするには、対応するドライバをダウンロードしてインストールする必要があります。

1) デバイスがホストマシンに接続されていないことを確認します。
2) macOS 用のSilicon Labs CP210x driverを[こちら](https://www.silabs.com/documents/public/software/Mac_OSX_VCP_Driver.zip) ダウンロードします。
3) ダウンロードしたファイルを解凍し、 **SiLabsUSBDriverDisk.dmg** を実行します
4) マウントされると **Install C210x VCP Driver** がありますので、実行してインストールを行います
   - macOS 10.13 もしくはより新しいバージョンではインストールがブロックされる場合があります。その場合は、 **System Preferences** <i class="fas fa-arrow-right"></i> **Security & Privacy** と開き、 鍵アイコン <i class="fas fa-lock"></i> をクリックして解除し、開発元を  **許可** し、再度 鍵アイコン <i class="fas fa-lock-open"></i> をクリックします。詳しくは [Apple Technical Note TN2459](https://developer.apple.com/library/archive/technotes/tn2459/_index.html)を参照してください。
5) コンピュータを再起動すると利用の準備が整います

## Visual Studio Code のインストール
Visual Studio Code (VS Code) は、コードを表示、編集、管理できるオープンソースの統合開発環境(IDE) です。お使いのオペレーティングシステムの最新の [Visual Studio コード](https://code.visualstudio.com/) をダウンロードします。Visual Studio コードのインストールまたは使用に関する問題のトラブルシューティングについては、[それらのドキュメント](https://code.visualstudio.com/docs/setup/setup-overview) を参照してください。

{{% notice note %}}
ダウンロードしたVS Codeは **Downloads** フォルダでそのまま利用するのではなく、**Application** フォルダに移動してから利用することを推奨します。
{{% /notice %}}

## PlatformIOのインストール
[PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide)(PIO) は、組み込みソフトウェア開発を簡素化するプロフェッショナルな組み込み開発プラットフォームを提供します。VS Codeの拡張機能は、PlatformIO コマンドラインインターフェイス(CLI) のグラフィカルインターフェイスで機能を提供します。拡張機能のダウンロードについては、[こちら](https://platformio.org/install/ide?install=vscode)を参照してください。

PlatformIOの拡張機能をインストールしたら、VS Codeを再起動する必要があります。

## ソースコードの取得
プロジェクトとファイルはすべて [GitHub リポジトリ](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/about-repositories) に存在し、リポジトリ内の各ファイルのリビジョン履歴を表示することもできます。チュートリアルに必要なコードをコピーするには、次の方法でPIOインターフェースを使用します。

1. VS Codeアクティビティバー（左端のメニュー）でPlatformIOのロゴをクリックする。
2. PlatformIO の **Quick Access**メニューから、**Miscellaneous** で、**Clone Git Project**を選択します。
3. テキストフィールドに `https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git` を貼り付けて、プロジェクトを保存する場所を選択します。
{{< img "pio-clone_git_project.en.png" "PlatformIO Clone Git Project" "1 - PIOメニューを開く。 2 - Clone Git Projectを選択, 3 - リポジトリURLを貼り付け" >}}

## スマートフォン向けアプリのダウンロードとインストール
ESP RainMaker Phone Apps は、iOS および Androidで Wi-Fi ネットワークの構成、ユーザー作成、ユーザーデバイスの関連付け、およびデバイス制御を提供するために利用できます。アプリはこちらにあります：
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker)
  * 直接APKを取得する場合[こちらから](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

互換性のある Android または iOS デバイスを所有していない場合は、[RainMaker CLI](https://rainmaker.espressif.com/docs/cli-setup.html) を使用することもできます。使い方についてはサイトの[CLI Usage](https://rainmaker.espressif.com/docs/cli-usage.html)を参照してください。

## USBポートの確認
Core2 for AWS IoT EduKitリファレンスハードウェアは同梱のUSB-C - USB-Aケーブルを利用して接続しますが、Mac側にUSB-Aポートが無い場合は、変換アダプターなどを別途用意したうえで利用してください。接続できたら電源を入れます。ホストマシンに接続すると、デバイスの電源が自動的にオンになりますが、電源を入れる必要がある場合は、電源ボタンを押します。
![AWS の M5Stack Core2 をオンまたはオフにする方法](linux/core2foraws_power_on_off.jpg?width=500px&classes=shadow)

デバイスの準備ができて、このチュートリアルに必要なソフトウェアをインストールしたら、デバイスが仮想的にマウントされているポートを特定して、そのポートに対する読み取りおよび書き込み操作を実行できるようにします。

1. PlatformIOの **Quick Access** メニューの **PIO Home** で、**Devices** を選択します
2. Descriptionの列で `CP2104 USB to UART Bridge Controller`と書かれた行が見つかりますので、Port列ののパスが書かれている右側に表示されているアイコンをクリックして、パスをコピーします。(通常は `/dev/cu.SLAB_USBtoUART`)

{{% notice note %}}
リストに表示されない場合は、電源が入っていること、および付属の USB-A - USB-C ケーブルを使用していることを確認してください。USB-C ハブの中には、シリアルポートの確立に互換性の問題があるものがあります。
{{% /notice %}}

## 次の手順
すべてのセットアップとホストマシンの準備ができて、Core2 for AWS IoT EduKitリファレンスハードウェアと通信できる状態になったら、次の章 [**ESP RainMaker エージェントの実行**](/jp/getting-started/run-rainmaker.html) に進みます。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}