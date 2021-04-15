+++
title = "まとめ"
weight = 60
pre = "<b>f. </b>"
+++

## まとめ
スマートサーモスタットとしてデバイスを AWS に接続し、簡単なアプリケーションをデプロイして、部屋の在室状況を検知し、HVAC の状態を制御する、AWS IoT EduKit の実践的チュートリアルが完了しました。

次のステップとしては、このシリーズの次の実践的チュートリアル、 「スマートスペース」の受講を検討してください。

新しいソリューションでテストするアイデアを検討している場合、カスタマーエクスペリエンスを強化する方法を考えてみてください。例えば、たった 10 秒間雑音が聞こえなかったらオフになるのではなく、HVAC 設定が最小タイマーでエンゲージされるようにソリューションをアップデートするにはどうすればいいでしょうか? デバイスに新しいコードをプッシュすることなく、実現することは可能ですか? (ヒント: 可能です)

## クリーンアップ
このソリューションでは、AWS で次のリソースを作成しました

* IoT Core ルール
* IAM ロール
* IoT Events 入力および探知器モデル

これらのリソースで、現在進行形の従量制課金は発生しません。デバイスからの新しいメッセージを処理するためにリソースが使用されたときのみ、従量制課金が発生します。次の実践的チュートリアル、「スマートスペース」に移動する場合、このチュートリアルで作成したリソースは破棄しないことをお勧めします。

このチュートリアルのソリューションにてテストを完了し、「スマートスペース」に進まない場合は、これらのリソースを破棄することができます。

1. AWS IoT Core に移動し、[Act] を選択してから [Rules (ルール)] を選択し、リストからルールを見つけて削除します。
1. AWS IoT Events に移動して、[Detector models (探知器モデル)] を選択し、リストからモデルを見つけて削除します。[Inputs (入力)] を選択し、リストから入力を見つけて削除します。
1. IAM に移動し、[Rules (ルール)] を選択して、リストからIoT Core ルール向けのロールと IoT Events 探知器モデルを見つけて削除します。
1. [PlatformIO CLIターミナルウィンドウ](/jp/blinky-hello-world/prerequisites.html#platformio) の **Smart-Thermosat** フォルダで以下のコマンドを実行して、デバイスのファームウェアを消去することで、デバイスが AWS IoT Core に接続しメッセージを送信できないようにします。電源ボタンを6秒間押し続けてデバイスの電源を切ることもできます。

```bash
pio run --environment core2foraws --target erase
```

次のチュートリアルは、[スマートスペース](/jp/smart-spaces.html)です

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}