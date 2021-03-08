+++
title = "Lチカ"
weight = 40
pre = "<b>d. </b>"
+++

この時点で、デバイスソフトウェアは実行され、AWS IoT Core に接続されて、メッセージを送信しています。また、クラウドからメッセージを受信したり対応したりする準備ができています。この章では MQTT トピックにサブスクライブし、AWS IoT Core が受信するメッセージを表示したり、メッセージを送信してデバイス LED を点滅させたりします。MQTT はパブリッシュ/サブスクライブプロトコルで、特定のトピックにサブスクライブしたり、パブリッシュしたりできます。登録スクリプトで以前設定されていたポリシーは、デバイスがサブスクライブおよびパブリッシュできるトピックを制限していました。これはフィルタリングと、潜在的なセキュリティの観点において重要です。デバイスはクライアント ID に一致するトピックのみを送信または受信できます。この場合クライアント ID とは、デバイスのセキュアエレメントの固有シリアル番号と同じです。

## AWS IoT MQTT クライアント経由でのサブスクライブ
[AWS IoT Core コンソール](https://us-west-2.console.aws.amazon.com/iot/home?region=us-west-2#/)の AWS IoT MQTT クライアントを使用すると、MQTT メッセージを表示およびパブリッシュができます。開始するには、AWS IoT コンソールに移動し、[Test (テスト)] を選択してクライアントビューを開きます。

{{< img "choose-test.png" "Choose test in AWS IoT console" >}}

[Subscription topic (トピックのサブスクリプション)] フィールドに`#`を入力して、[Subscribe to topic(トピックへのサブスクライブ)]のボタンを選択すると、すべての MQTT トピック名にサブスクライブします。マルチレベルのワイルドカード[トピックフィルター](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#topicfilters)は「#」で、トピックフィルターの最後の文字として 1 度のみ使用できます。[Subscribe to topic (トピックにサブスクライブ)] ボタンを押すと、デバイスから送信されるメッセージが表示されます。デバイスは `<<CLIENT_ID>>/` から始まるトピックにメッセージをパブリッシュすることのみ許可されています。これにより、他のサブスクライバー (クラウドアプリケーションなど) が特定のクライアントデバイスのトピックをフィルターできるようになり、範囲をより限定的なトピックに狭めることもできるようになります (特定デバイスでの温度読み上げなど)。

## LEDの点滅
M5Stack Core2 for AWS IoT EduKit リファレンスハードウェアの側面にある LED バーの点滅を開始または停止するため、コンソールの AWS IoT MQTT クライアントから、Core2 for AWS IoT EduKit がサブスクライブしているトピックをパブリッシュします。そのためにはまず、デバイスのクライアント ID を取得する必要があります。シリアルモニターがまだ実行中のシェルプロンプトに戻り、**クライアント ID** をコピーします。

{{% notice note %}}
パスに ESP-IDF が追加され、Conda edukit 環境がアクティブとなった状態(シェルプロンプトに**(edukit)**が表示されている)であることを確認してください。シェルを閉じたり、新しいシェルを開いたりする場合は、毎回 `conda activate edukit` を再入力して、環境を再起動し、ESP-IDF toolをパスに追加するために、`export.sh` (macOS/Linux) or `export.bat` (Windows) を実行します。
{{% /notice %}}

[Publish (発行)] ボックスで以下のコマンドを入力します。ただし、**<<CLIENT_ID>>** テキストは、先ほどコピーした実際のクライアント ID に置き換えます。それから [Publish to topic (トピックに発行)] ボタンを押します。
```
<<CLIENT_ID>>/blink
```
{{< img "aws_iot_mqtt_client.png" "Subscribing to messages and publishing with AWS IoT console MQTT client" >}}
デバイスのサイドバーの LED が点滅しているはずです。点滅を一時停止するには、もう一度同じトピックの [Publish to topic (トピックに発行)] ボタンを押すだけです。LED を点滅させ続けていた点滅タスクが一時停止されます。これらのタスクは FreeRTOS カーネルの一部 (スレッドでもある) で、単一のタスクを繰り返し実行するようになっています。FreeRTOS カーネルは、他のタスクが一時停止またはブロック状態になったときに (**vTaskSuspend** または **vTaskDelay** を使用) 個別のタスクが実行されるようスケジュール設定することで、マイクロコントローラーアプリケーションによるプロセッサ最適化を実現します。FreeRTOS を使用したスケジュール設定について詳しくは、[こちら](https://www.freertos.org/implementation/a00005.html)をご覧ください。

{{% notice info %}}
シリアルモニターを終了するには **CTRL** + **]** を押します。
{{% /notice %}}

## クリーンアップ
使用するリソースの最適化と不要な AWS クラウドのサービス料金を回避するため、デバイスのフラッシュを削除して、以降のチュートリアルに備えます。フラッシュを削除するには、すでにビルドされたプロジェクト (先ほど完了したものなど) に入り、コマンドを使用する必要があります。シリアルモニターが利用中だとポートがブロックされているため、**CTRL** + **]** を押してシリアルモニターを終了します。
```
idf.py erase_flash -p <<DEVICE_PORT>>
```

## まとめ

クラウドに接続されたLチカプロジェクトを楽しんでいただけたでしょうか。Espressif のツールチェーン (ESP-IDF) を使用して、Core2 for AWS IoT EduKit のファームウェアをビルド、フラッシュ、モニタリングすることができました。デバイスは AWS IoT のモノとして AWS に登録されました。埋め込み型デバイス証明書を使用したセキュアなクラウド接続が確立され、デバイスを決して離れることのないプライベートキーが提供されます。AWS IoT Core に接続し、MQTT 経由でデバイスからメッセージを送信し、デバイスのライトを切り替えるために使用された AWS IoT コンソールの MQTT クライアントから MQTT メッセージを受信しました。

次のチュートリアル、[スマートサーモスタット](/ja/smart-thermostat.html)に移りましょう。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}