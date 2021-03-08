+++
title = "前提条件"
weight = 11
pre = "<b>a. </b>"
+++

この章では、バイナリファームウェアの表示、編集、コンパイルに必要なソフトウェアをダウンロードしてインストールし、ファームウェアをデバイスに転送して実行します。さらに、デバイスを制御するためにスマートフォンにアプリをダウンロードします。

## Silicon Labs USB to UART ブリッジドライバのインストール
SiLabs CP210xドライバーをダウンロードしてインストールし、コンピューターがCore2 for AWS IoTEduKitデバイスと通信できるようにします。 オンボードのCP2104は、ESP32-D0WDマイクロコントローラーとのホスト通信するUSB-UARTブリッジです。

{{%expand "macOS 10.9+" %}}
OS X Mavericks 以降、Appleはシリアルドライバに必要なUSBを組み込んでおり、追加の手順は必要ありません。 ドライバがインストールおよびロードされていることを確認するには、次のコマンドを実行します。
```bash
kextstat | grep usb.serial
```
実行結果に、`com.apple.driver.usb.serial`が存在する場合、ロードされたアドレスと合わせて表示されます。 上記の`コマンドの結果が空を返す場合`は、macOS <= 10.8の手順に従ってください。
{{% notice info %}}
macOS 10.13以降を利用している場合、ドライバのインストールがブロックされる場合があります。ブロックされた場合は、Macの **システム環境設定** <i class="fas fa-arrow-right"></i> **セキュリティーとプライバシー** と開き、**一般** のタブを開きます。ブロックされているメッセージが表示されていれば、左下の<i class="fas fa-lock"></i>を選択してロックを解除し、許可ボタンを選択し、<i class="fas fa-lock-open"></i>を選択してロックします。詳しくは、[Apple Technical Note TN2459](https://developer.apple.com/library/archive/technotes/tn2459/_index.html)を参照してください。
{{% /notice %}}
{{% /expand%}}
{{%expand "macOS <= 10.8" %}}
1. Core2 for AWS IoT EduKitデバイスが既に接続されている場合は、コンピューターに接続しているUSB-Cケーブルを外します
2. [CP210x VCPmacOSドライバー](https://www.silabs.com/documents/public/software/Mac_OSX_VCP_Driver.zip)をダウンロードして解凍します
3. 抽出したフォルダー内から**SiLabsUSBDriverDisk.dmg**ファイルを展開します。
4. **Install CP210x VCP Driver**アプリケーションを開き、インストーラーを実行します
5. コンピューターを再起動し、付属のUSB-Cケーブルを接続してCore2 for AWS IoTEduKitデバイスをコンピューターに再接続します
{{% /expand%}}
{{%expand "Linux" %}}
Linuxカーネルバージョン3.x.xおよび4.x.xには、ディストリビューションの一部としてすでにドライバーが含まれています。 それらがインストールおよびロードされていることを確認するには、ターミナルで次のコマンドを実行します。
```bash
modinfo usbserial
modinfo cp210x
```
**ロードされていない場合**は、次のコマンドを実行します。
```bash
sudo modprobe usbserial
sudo modprobe cp210x
```
{{% /expand%}}
{{%expand "Windows" %}}
1. Core2 for AWS IoT EduKitデバイスが既に接続されている場合は、コンピューターに接続しているUSB-Cケーブルを外します
2. [CP210xユニバーサルWindowsドライバー](https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip)をダウンロードして解凍します
3. **CP210xVCPInstaller_x64.exe**を開き、インストーラーを実行してドライバーをインストールします
4. コンピューターを再起動し、付属のUSB-Cケーブルを接続してCore2 for AWS IoTEduKitデバイスをコンピューターに再接続します
{{% notice warning %}}
チュートリアルは、Windows10 64ビットでのみテストされています。 Windowsオペレーティングシステムの他の構成はサポートしていません。
{{% /notice %}}
{{% /expand%}}

## Visual Studio Code のインストール
Visual Studio Code は、コードの閲覧、編集、管理などを行えるオープンソースの統合開発環境 (IDE) です。使用しているオペレーティングシステム用の最新の [Visual Studio Code](https://code.visualstudio.com/) をダウンロードします。Visual Studio Code のインストールや使用方法に関するトラブルシューティングについては、Visual Studio Code の[ドキュメント](https://code.visualstudio.com/docs/setup/setup-overview)を参照してください。

## PlatformIO のインストール
[PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) は、組み込みソフトウェアの開発を簡素化するプロフェッショナルな組み込み開発プラットフォームを提供します。Visual Studio Code 拡張機能を使用すると、PlatformIO コマンドラインインターフェイス (CLI) の機能をグラフィカルインターフェイスで使用できるようになります。拡張機能のダウンロードと、PlatformIO の詳細の確認については、[こちら](https://platformio.org/install/ide?install=vscode)をご覧ください。

{{%expand "Windows" %}}
{{% notice info %}}
すでにEspressif IoT Development Framework (ESP-IDF)がインストールされたコンピューターをご利用の場合、**esp-windows-curses** のファイルが存在しないや、依存関係を解決できない様なエラーが表示される場合があります。この問題を解決する方法は、PlatformIOコミュニティーの[こちらの投稿](https://community.platformio.org/t/cant-create-esp-idf-project-correctly-in-platformio/16370/17)を参考にしてください。
{{% /notice %}}
{{% /expand%}}

## スマートフォンアプリのダウンロードとインストール
ESP RainMaker スマートフォンアプリは、iOS および Android のスマートフォン向けに提供されており、Wi-Fi ネットワークの設定、ユーザー作成、ユーザーとデバイスの関連付け、デバイスの制御などを行うことができます。アプリは次の場所にあります
* Android: [Google PlayStore](https://play.google.com/store/apps/details?id=com.espressif.rainmaker), [Direct APK](https://github.com/espressif/esp-rainmaker-android/releases)
* iOS: [Apple App Store](https://apps.apple.com/app/esp-rainmaker/id1497491540)

互換性のある Android や iOS デバイスをお持ちでない場合は、[Rainmaker CLI](https://rainmaker.espressif.com/docs/cli-setup.html)を使用して指示を代用することができます。

## コードのダウンロード
このプログラムで使用するコードをダウンロードするには、[GitHub](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit) でホストされているリポジトリをcloneするか、[zip](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/archive/master.zip) ファイルをダウンロードして抽出します。Gitでcloneするには:

```bash
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```
{{%expand "Windows" %}}
{{% notice info %}}
Windowsにはファイルパスの最大長に関する制限があります。深い階層にダウンロードしたzipファイルを展開したり、git cloneするとこのエラーが発生する場合があります。なるべく浅いディレクトリに配置してください。(例:**C:\\**)
{{% /notice %}}
{{% /expand%}}

## デバイスの接続と電源オン
最後に、デバイスがパソコンに接続され、電源が入っていることを確認してください。デバイスを接続して電源を入れれば自動的に電源が入りますが、オンにする必要がある場合は、電源ボタンを押してください。
![How to turn M5Stack Core2 for AWS on or off](prerequisites/core2foraws_power_on_off.jpg?width=500px&classes=shadow)

デバイスの準備ができたら、次の章[**ESP RainMaker Agent の実行**](/ja/getting-started/run-rainmaker.html)に進みましょう。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}