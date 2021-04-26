+++
title = "デバイスのプロビジョニング"
weight = 20
pre = "<b>b. </b>"
+++

この章では、オンボードの Microchip ATTECC608 Trust&GO セキュアエレメントを使用して AWS IoT Core に接続するためのデバイスをプロビジョニングし、[TLS 接続](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html)を確立します。組み込みのハードウェアの信頼の基点により、プライベートキーを公開することなく、簡素化された迅速なプロビジョニングパスを持つことができます。デバイスに組み込まれているデバイス証明書を取得し、マニフェストファイルを作成して AWS IoT のモノ (デバイスの表現と記録) を作成することができます。このデバイスのクライアント ID は、AWS IoT Core に登録され、シリアル番号で識別されます。同様のプロセスを使用することによって、一度に何千、何百万ものデバイス群のデプロイを自動化することができます。

## Lチカ(Blink Hello World)のプロジェクトを開きます

VS Codeで既に開いている他のプロジェクトがある場合は、まず新しいウィンドウ（**File**→**New Window**）を開き、ファイルエクスプローラと作業環境を作成します。

このチュートリアルでは、Blinky-Hello-World プロジェクトを使用します。新しいVS Codeのウィンドウで、**PlatformIOロゴ** VS Codeアクティビティバー(左側のメニュー) をクリックし、左の PlatformIO メニューから**Open** を選択し、**Open Project** をクリックし、`Core2-for-AWS-IOT-edukit/Blinky-Hello-World`フォルダに移動し、**open**をクリックします。
{{< img "pio-home.en.png" "PlatformIO home screen" "1 - PIOメニューを開く。 2 - PIO Homeを開く。 3 - プロジェクトフォルダを開く" >}}

プロジェクトを開いたら、PlatformIO CLI ターミナルウィンドゥも開きます。

1. VS Codeのアクティビティーバー(左のメニュー)で **PlatformIOのロゴ** をクリックします
2. **Quick Access** メニューの、**Miscellaneous** にある **New Terminal** をクリックしてターミナルを開きます

## デバイス証明書の取得とAWS IoTへthingの登録

[MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html) 経由で AWS IoT Core への安全な TLS 接続を実行するには、モノを登録し、[デバイス証明書](https://docs.aws.amazon.com/iot/latest/developerguide/register-device-cert.html)を Thing に登録し、[セキュリティポリシー](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) を証明書に紐付けて、AWS アカウント内で不正なデバイスや不正な操作が実行されないようにします。Core2 for AWS IoT EduKitファレンスハードウェアにセキュアエレメント利用しているため、機密性の高い秘密鍵を公開したり処理したりすることなく、デバイス登録全体を自動化できます。

プロジェクトには、ハードウェアのセキュアエレメントから事前にプロビジョニングされたデバイス証明書を取得する手順を自動化し、[X.509 証明書](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html#x509-client-cert-basics)でデバイス証明書に署名してデバイスマニフェストを生成するスクリプトが含まれています。Microchip(セキュアエレメントのメーカー) の [ジャストインタイム登録](https://aws.amazon.com/blogs/iot/just-in-time-registration-of-device-certificates-on-aws-iot/) を行い、AWS IoT はデバイス証明書を使用して、AWS IoT Thing に安全なポリシーをアタッチします。


提供されているスクリプトを、PlatformIO CLI ターミナルウィンドゥで実行する方法については、利用しているOS毎に異なりますので、以下の対象となる説明を開いて作業を進めます。

{{% notice warning %}}
過去にこのセットアップをされている場合、すでにAWS IoT Coreに情報が登録されている可能性があります。その場合は、ターミナルに表示される `Unique ID: 0123456789ABCD` の様なIDの名前のモノや証明書、それに紐づくIoT PolicyをAWS IoT Coreのマネージメントコンソールから削除してから、再度実行してください。
{{% /notice %}}

{{%expand "Ubuntu または macOS" %}}
レジストレーションスクリプト( **utilities/AWS_IoT_registration_helper** に存在)を実行するには、以下のコマンドをPlatformIO CLIターミナルで実行します。実行する際には、**<<DEVICE_PORT>>** には実際にデバイスが接続されているポートを指定します。デバイスが接続されているポートを確認するには、[Ubuntu](../getting-started/prerequisites/linux.html#usb) または [macOS](../getting-started/prerequisites/macos.html#usb)を参照してください。

```bash
cd utilities/AWS_IoT_registration_helper
python3 registration_helper.py -p <<DEVICE_PORT>>
```

デバイスが正しくAWS IoTに登録されたら、以下のコマンドで **Blinky-Hello-World** ディレクトリに移動します。
```bash
cd ../..
```
{{% /expand%}}


{{%expand "Windows" %}}
レジストレーションスクリプト( **utilities¥AWS_IoT_registration_helper** に存在)を実行するには、以下のコマンドをターミナルで実行します。実行する際には、**<<DEVICE_PORT>>** には実際にデバイスが接続されているポートを指定します。デバイスが接続されているポートを確認するには、[Windows](../getting-started/prerequisites/windows.html#usb)を参照してください。

```PowerShell
cd utilities/AWS_IoT_registration_helper
python registration_helper.py -p <<DEVICE_PORT>>
```

デバイスが正しくAWS IoTに登録されたら、以下のコマンドで **Blinky-Hello-World** に移動します。
```PowerShell
cd ../..
```
{{% /expand%}}

## 章のまとめ
この章では、セキュアエレメントを使用して AWS IoT の[モノ](https://docs.aws.amazon.com/iot/latest/developerguide/thing-registry.html)を作成し、モノ用の[IoTポリシー](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html)を設定し、モノに[デバイス証明書](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html)をアタッチしました。セキュアエレメントを利用することで、プライベートキーを公開したりすることなく安全にプロビジョニングが実行されます。

「[AWS IoT Core への接続](/jp/blinky-hello-world/connecting-to-aws.html)」に進みます。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}