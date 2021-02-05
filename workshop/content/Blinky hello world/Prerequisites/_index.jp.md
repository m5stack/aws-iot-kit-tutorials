+++
title = "前提条件"
weight = 10
pre = "<b>a. </b>"
+++

この章では、他の章でも必要となる、必須のソフトウエアのインストールを行います。最初にCore 2 for AWS IoT EduKitをUSBで利用するために必要なSiLabs CP210xドライバをインストールし、Espressif ESP32-D0WD マイクロコントローラーユニット (MCU)の設定、ビルド、フラッシュを行うために、ESP-IDF (Espressif IoT Development Framework)のインストールを行います。また、Miniconda をインストールして、隔離された仮想環境の Python のバージョンと依存関係を管理し、競合を回避します。Jupyter をインストールして、AWS IoT のモノの登録に使用するノートブックを実行します。さらに、AWS コマンドラインインターフェイス (CLI) をダウンロードして、インストールおよび設定します。このチュートリアルは、[AWS アカウント](https://signin.aws.amazon.com/signin)を持っていることを前提とします。

## Silicon Labs USB to UART ブリッジドライバーのインストール
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

## ESP-IDF v4.2 のインストール
お使いの **M5Stack Core2 for AWS2** リファレンスハードウェアには、Espressif ESP32-D0WD マイクロコントローラーユニット (MCU) が搭載されています。ESP-IDF (Espressif IoT Development Framework) を使用すると、ESP32 ボード上でファームウェアを設定、ビルド、フラッシュすることができます。「開始方法」チュートリアルでは、PlatformIO を使用して、デバイスのファームウェアのコンパイルとフラッシュを簡素化しました。PlatformIO には、ESP-IDF バージョン 4.1 やその他のツールにバックグラウンドで呼び出しを実行できる GUI が備わっています。以降のステップでは、mbedTLS ライブラリの強化とセキュア要素の統合の改善から、ESP-IDF バージョン 4.2 を使用します。

{{%expand "macOS" %}}
1) まだの場合は、Homebrew をインストールします(インストールに時間がかかる場合があります)
2) cmake、ninja、DFU ユーティリティをインストールします。
3) ESP-IDF を `$HOME/esp/` ディレクトリにcloneします (ESP-IDF のフルパスは **$HOME/esp/esp-idf** です)。
4) ESP-IDF の依存関係をインストールします。
5) インストールを確認し、ESP-IDF 環境変数を **$PATH** にエクスポートします。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install cmake ninja dfu-util
mkdir $HOME/esp
cd $HOME/esp
git clone -b release/v4.2 --recursive https://github.com/espressif/esp-idf.git
cd $HOME/esp/esp-idf
. $HOME/esp/esp-idf/install.sh
. $HOME/esp/esp-idf/export.sh
```
{{% /expand%}}
{{%expand "Linux" %}}
1) すべての依存関係をインストールします
   - CentOS 7:
   ```bash
   sudo yum install git wget flex bison gperf cmake ninja-build ccache dfu-util
   ```
   - Ubuntu と Debian:
   ```bash
    sudo apt-get install git wget flex bison gperf python-setuptools cmake ninja-build ccache libffi-dev libssl-dev dfu-util
    ```
    - Arch:
    ```bash
    sudo pacman -S --needed gcc git make flex bison gperf cmake ninja ccache dfu-util
    ```
2) ESP-IDF を `$HOME/esp/` ディレクトリにcloneします (ESP-IDF のフルパスは **$HOME/esp/esp-idf** です)。
3) ESP-IDF の依存関係をインストールします。
4) インストールを確認し、ESP-IDF 環境変数を **$PATH** にエクスポートします。

```bash
mkdir $HOME/esp

cd $HOME/esp

git clone -b release/v4.2 --recursive https://github.com/espressif/esp-idf.git

cd $HOME/esp/esp-idf

. $HOME/esp/esp-idf/install.sh

. $HOME/esp/esp-idf/export.sh
```
{{% /expand%}}
{{%expand "Windows (64-bit)" %}}
1) [ESP-IDF Tools Installer](https://dl.espressif.com/dl/esp-idf-tools-setup-2.3.exe)をダウンロードして使用し、**ESP-IDF v4.2** とその他のすべての依存関係をインストールします。
2) インストーラーの実行後、まだ実行していない場合は、マシンを再起動してください。
3) `cd %userprofile%\esp\esp-idf`
4) ESP-IDF のインストールスクリプトを実行して、すべてが正しく設定されていることを確認してください: `install.bat`
5) `export.bat`を実行して、ESP-IDF をパスに追加します
{{% /expand%}}

## AWS CLI バージョン 2 のインストールと設定
### AWS CLI インストール
AWS コマンドラインインターフェイス (CLI) は、AWS のサービスを管理するための統合ツールです。ダウンロードおよび設定用の単一のツールのみを使用して、コマンドラインから複数の AWS サービスを制御し、スクリプトを使用してこれらを自動化することができます。AWS CLI を設定するには、最初に AWS アカウントが必要です。[サインイン](https://signin.aws.amazon.com/signin)するか、[アカウントを作成](https://portal.aws.amazon.com/billing/signup#/start)してから続行してください。サインインしたら、使用している OS 用の公式の [AWS CLI のインストール手順](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)に従います。


### AWS CLI 設定
インストールしたら、アカウントとリージョンに合わせて設定します。常に同じリージョンを使用するように注意してください。このチュートリアルでは、**us-west-2** に統一してあります。リージョンを変更すると、その後のステップで別の問題が発生することがあります。リージョナルサービスの可用性を確認するには、[AWS リージョナルサービスリスト](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)を参照して、利用可能なサービスの一覧を確認します。

次のコマンドを使用します。
```bash
aws configure
```
AWS CLI の認証情報を取得する手順については、[クイック設定](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)ドキュメントを参照してください。AWS アクセスキー ID と AWS シークレットアクセスキーを取得するには、[IAM ユーザーに必要な許可](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)を与える必要があります。入力は次のようになります。
```
AWS Access Key ID [None]: EXAMPLEKEYIDEXAMPLE
AWS Secret Access Key [None]: EXAMPLEtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## Miniconda のセットアップとインストール
Miniconda は、本格的な Anaconda のブートストラップ版であり、パッケージと Python の仮想環境管理を簡素化します。
多くのインストールエラーはPythonのバージョン(e.g. Python 2.x や 3.8.x)が異なることや、pythonのPATHが異なるバージョンを参照していることから発生します。この様な問題を回避するためにも、Condaを利用することを**推奨**します。
https://docs.conda.io/en/latest/miniconda.html を訪問し、利用しているOS向けのインストーラーをダウンロードしてください。公式のセットアップ手順を終了したら、こちらのページに戻ります。

{{%expand "macOS and Linux" %}}
1) minicondaのインストール後、作業していたターミナルソフトを一度終了し、再度開くことで、必要なファイルが追加され、環境変数が設定されます。
2)  conda 環境を新規で作成することで環境を分離し、依存関係の競合を回避します。以下のコマンドを実行し、**y**を入力することで、必要な依存関係がインストールされます
   ```bash
   conda create -n edukit python=3.7
   ```
3) 新しい環境の作成に成功したら、base 環境から切り替えます。現在のプロンプトの先頭に (edukit) と表示されるため、アクティブになっていることがわかります。
   ```bash
   conda activate edukit
   ```
4) ESP-IDF toolchain export scriptを実行することで、これらのツールがパスに追加されます
    ```bash
    . $HOME/esp/esp-idf/export.sh
    ```
{{% /expand%}}
{{%expand "Windows (64-bit)" %}}
1) minicondaのインストーラーが終了したら、windowsを再起動を行います
2) Anacondaのプロンプトは、**start menu** → **Anaconda prompt** より開きます
3) conda 環境を新規で作成することで環境を分離し、依存関係の競合を回避します。以下のコマンドを実行し、**y**を入力することで、必要な依存関係がインストールされます
   ```bash
   conda create -n edukit python=3.7
   ```
4) 新しい環境の作成に成功したら、base 環境から切り替えます。現在のプロンプトの先頭に (edukit) と表示されるため、アクティブになっていることがわかります。
   ```bash
   conda activate edukit
   ```
5) スクリプトを実行して、ESP-IDF toolsをパスに追加します
   ```bash
   %userprofile%\Desktop\esp-idf\export.bat
   ```
{{% /expand%}}

{{% notice note %}}
シェルを閉じたり、新しいシェルを開いたりする場合は、毎回 `conda activate edukit` を再入力して、環境を再起動し、ESP-IDF toolをパスに追加するために、`export.sh` (macOS/Linux) or `export.bat` (Windows) を実行します。
{{% /notice %}}

## コードのダウンロード
[開始方法 > 前提条件](/ja/getting-started/prerequisites.html)の手順でソースコードを間違えた場所に置いた場合、[GitHub](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit) でホストされているリポジトリをcloneするか、[zip](https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/archive/master.zip) ファイルをダウンロードして抽出します。Gitでcloneするには:

```bash
git clone https://github.com/m5stack/Core2-for-AWS-IoT-EduKit.git
```
{{%expand "Windows" %}}
{{% notice info %}}
Windowsにはファイルパスの最大長に関する制限があります。深い階層にダウンロードしたzipファイルを展開したり、git cloneするとこのエラーが発生する場合があります。なるべく浅いディレクトリに配置してください。(例:**C:\\**)
{{% /notice %}}
{{% /expand%}}

すべてのインストールと設定が完了したら、次の章「[デバイスのプロビジョニング](/ja/blinky-hello-world/device-provisioning.html)」に移動してください。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}