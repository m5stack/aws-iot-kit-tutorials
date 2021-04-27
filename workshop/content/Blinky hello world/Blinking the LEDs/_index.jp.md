+++
title = "Lチカ"
weight = 40
pre = "<b>d. </b>"
+++

この時点で、デバイスソフトウェアは実行され、AWS IoT Core に接続されて、メッセージを送信しています。また、クラウドからメッセージを受信したり対応したりする準備ができています。この章では [MQTT トピック](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html)にサブスクライブし、AWS IoT Core が受信するメッセージを表示したり、メッセージを送信してデバイス LED を点滅させたりします。MQTT は[パブリッシュ/サブスクライブプロトコル](https://mqtt.org/)で、特定のトピックにサブスクライブしたり、パブリッシュしたりできます。登録スクリプトで以前設定されていたポリシーは、デバイスがサブスクライブおよびパブリッシュできるトピックを制限していました。これはフィルタリングと、潜在的なセキュリティの観点において重要です。デバイスはクライアント ID に一致するトピックのみを送信または受信できます。この場合クライアント ID とは、デバイスのセキュアエレメントの固有シリアル番号と同じです。

## AWS IoT MQTT クライアント経由でのサブスクライブ
[AWS IoT Core コンソール](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/)の AWS IoT MQTT クライアントを使用すると、MQTT メッセージを表示およびパブリッシュができます。開始するには、AWS IoT コンソールに移動し、[Test (テスト)] を選択してクライアントビューを開きます。

{{< img "aws_iot-mqtt_test_client-subscribe.en.png" "Choose test in AWS IoT console" >}}

**Subscription topic (トピックのサブスクリプション)** フィールドに`#`を入力して、**Subscribe to topic(トピックへのサブスクライブ)** のボタンを選択すると、すべての MQTT トピック名にサブスクライブします。マルチレベルのワイルドカード[トピックフィルター](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#topicfilters)は「#」で、トピックフィルターの最後の文字として 1 度のみ使用できます。**Subscribe to topic (トピックにサブスクライブ)** ボタンを押すと、デバイスから送信されるメッセージが表示されます。デバイスは `<<CLIENT_ID>>/` から始まるトピックにメッセージをパブリッシュすることのみ許可されています。これにより、他のサブスクライバー (クラウドアプリケーションなど) が特定のクライアントデバイスのトピックをフィルターできるようになり、範囲をより限定的なトピックに狭めることもできるようになります (特定デバイスでの温度読み上げなど)。

## LEDの点滅
M5Stack Core2 for AWS IoT EduKit リファレンスハードウェアの側面にある LED バーの点滅を開始または停止するため、コンソールの AWS IoT MQTT クライアントから、Core2 for AWS IoT EduKit がサブスクライブしているトピックをパブリッシュします。このサブスクライブしているトピックは `<<CLIENT_ID>>/#` となります。デバイス側でサブスクライブが正しくされると、シリアルモニターにそのメッセージが出力されます。また、クライアントID(CLIENT_ID)も出力されます。

**Publish (発行)** ボックスで以下のコマンドを入力します。ただし、**<<CLIENT_ID>>** テキストは、先ほどコピーした実際のクライアント ID に置き換えます。それから **Publish to topic (トピックに公開する)** ボタンを押します。
```
<<CLIENT_ID>>/blink
```
{{< img "aws_iot-mqtt_test_client-publish.en.png" "Subscribing to messages and publishing with AWS IoT console MQTT client" >}}
デバイスのサイドバーの LED が点滅しているはずです。点滅を一時停止するには、同じトピックにメッセージを発行すれば停止できますので、**Publish to topic (トピックに発行)** ボタンを押すだけです。

{{% notice info %}}
シリアルモニターを終了するには **CTRL** + **C** を押します。
{{% /notice %}}

## クリーンアップ
使用するリソースの最適化と不要な AWS クラウドのサービス料金を回避するため、デバイスのフラッシュを削除して、以降のチュートリアルに備えます。フラッシュを削除するには、すでにビルドされたプロジェクト (先ほど完了したものなど) に入り、コマンドを使用する必要があります。シリアルモニターが利用中だとポートがブロックされているため、**CTRL** + **C** を押してシリアルモニターを終了します。シリアルモニターを停止したら、以下のコマンドでフラッシュを削除します

```
pio run --environment core2foraws --target erase
```

以上でこの章の手順は終了です。最後にこの章の[まとめ](conclusion.html)にに進みます。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}
