+++
title = "前提条件"
weight = 10
pre = "<b>a. </b>"
+++

この章では、ホストマシンのオペレーティングシステム用の [AWS CLI](https://aws.amazon.com/cli/) をダウンロードしてインストールし、AWS [Identity and Access Management](https://aws.amazon.com/iam/) (IAM)のコンソールで認証情報を取得して AWS CLIで使うように設定し、AWS CLI が正常に動作していることまでを確認します。
このチュートリアルでは、既に [AWS account](https://console.aws.amazon.com/console/home) アカウントをもっており、[環境のセットアップ](/jp/getting-started/prerequisites.html)を終了している想定でいます。終わっていない場合は、リンク先の手順を進めてください。すでに、AWS CLI(version1 または、version2)がセットアップ済みであれば、テストのセクションまで飛ばしてください。


## PlatformIOのターミナルを開く

[開始方法](/jp/getting-started.html)でPIOをインストールし、PIOのターミナルが使えるところまで確認しました。ここからの作業では、同じようにPIOのターミナルを利用して進めていきます。PIO ターミナルウィンドウには、標準のターミナル/コマンドプロンプトにないアプリケーションやライブラリが事前にロードされます。 

もし、VS Codeを閉じてしまった、もしくは、PIOのターミナルウィンドウを閉じてしまった場合は、VS Codeを開いてから以下の手順を実行してください。

1) VS Codeのアクティビティーバー(左のメニュー)で **PlatformIOのロゴ** をクリックします
2) **Quick Access** メニューの、**Miscellaneous** にある **New Terminal** をクリックしてターミナルを開きます。**PlatformIO CLI**というラベルが付いたターミナルが開くはずです

{{< img "pio-new_terminal-alexa_intro.en.png" "PlatformIO CLI terminal in VS Code" "1 - PIOメニューを開く。2 - 新しいPIOターミナルを開く。3 - 'PlatformIO CLI'ターミナルセッションが開かれていることを確認">}}

## AWS CLIのダウンロードとインストール
AWS Command Line Interface (CLI)は、AWSのサービスを管理するための統合ツールです。CLIをダウンロードし、設定を行えば、AWSのサービスをコマンドラインから操作することができ、様々な処理を自動化することができます。AWS CLIを利用するには、AWSのアカウントが必要になります。既にアカウントを持っている場合は、[マネージメントコンソールにサインイン](https://console.aws.amazon.com/console/home)します。AWSのアカウントを持っていない場合は、こちらから[アカウントの作成](https://portal.aws.amazon.com/billing/signup#/start)を行ってください。

AWS CLIのダウンロードは、以下を参考に進めてください。

{{%expand "Ubuntu Linux v18.0+ (64-bit)" %}}
   64-bit Linux向けのAWS CLI version 2をインストールします。
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```
{{% /expand%}}

{{%expand "MacOS 10.14+" %}}
1) macOS向けのAWS CLI version 2のインストーラーを[ダウンロード](https://awscli.amazonaws.com/AWSCLIV2.pkg)します
2) ダウンロードしたインストーラーをダブルクリックして、表示された手順の通りインストールを進めてください。すべてのユーザー向けにインストールすることを推奨しますが、もし自分のみに対してインストールした場合は、[こちら](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-mac.html#cliv2-mac-install-gui)を参考にsimlynk作成し、パスに追加してください。
{{% /expand%}}

{{%expand "Windows 10 (64-bit)" %}}
1) 最新版の[Windows向けのインストーラー](https://awscli.amazonaws.com/AWSCLIV2.msi)をダウンロードします。
2) ダウンロードしたインストーラーをダブルクリックして、表示された手順の通りインストールを進めてください。
{{% /expand%}}

## IAM ユーザーアクセス認証情報を取得する

[IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)はAWSのリソースを利用する上でのアクセス権限を管理できるサービスです。Rootアカウントを利用するのではなく、[Administrator Access権限を持つユーザーを作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html)して利用することをおすすめします。

{{% notice warning %}}
AWS アカウントのセキュリティーベストプラクティスについては[こちら](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/best-practices.html)のページを参照してください。
{{% /notice %}}

IAMユーザーのクレデンシャルの取得方法については、[こちらのドキュメント](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config)を参考に取得してください。


## AWS CLIの設定

AWS CLI がインストールされ、IAM ユーザーアクセス認証情報が取得できたら、AWS CLI を設定します。設定する項目の1 つは AWS [リージョン](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) です。このチュートリアルでは、**us-west-2** を利用しています。別のリージョンを利用する場合、チュートリアルで利用するサービスによっては、提供されていない可能性がありますので、他のリージョンを利用する場合は、[AWS リージョン別のサービス](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)で対応しているかを確認してください。

ホストマシンで AWS CLI を設定するには、ターミナルビューポートで次のコマンドを入力します。
```bash
aws configure
```

CLIの設定では、4 つのパラメータの入力を要求するプロンプトが表示されます。フィールドは以下のように入力する必要があります。対応するアクセスキー ID とシークレットアクセスキーには、IAM ユーザー用に以前に取得したアクセスキー ID とシークレットアクセスキーを指定します。
```bash
AWS Access Key ID [None]: EXAMPLEKEYIDEXAMPLE
AWS Secret Access Key [None]: EXAMPLEtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

## AWS CLIの動作確認
すべて設定したら、AWS CLI が適切に動作していることを確認するために AWS CLI をテストします。まず、CLI 自体をテストして CLI がインストールされていることを確認します。次に、設定が有効であることをテストします。

CLIが正しくインストールされていることを確認するには、バージョンオプションを使用します。正常にインストールされると AWS CLI バージョンが出力されます。エラーが発生した場合は、[トラブルシューティングガイド](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)を参照してください。
```bash
aws --version
```

次に、AWS CLI が IAM 認証情報およびリージョンで設定されていることを確認する必要があります。実行するコマンドは、AWS IoT の [MQTT ブローカー](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) エンドポイントをチェックします。これは、`xxxxxx-ats.iot.us-west-2.amazonaws.com` の様な、アドレスを確認することができます。エラーが表示された場合は、[トラブルシューティングガイド](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)を参照してください。
```bash
aws iot describe-endpoint --endpoint-type iot:Data-ATS
```

すべてをインストールして設定したら、次の章 [**デバイスのプロビジョニング**](/jp/blinky-hello-world/device-provisioning.html) に進みましょう。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}