
+++
title = "まとめ"
weight = 50
pre = "<b>e. </b>"
+++

## まとめ
カスタム移植された Espressif Voice Assistant SDK と Alexa を使用して Alexa for AWS IoT (AFI) を実行し、Alexa スマートホームコマンドを使用してオンボード周辺機器を制御する、AWS IoT EduKit の実践的チュートリアルが完了しました。AVS 向けスマートホームの追加情報については、 [ドキュメンテーション](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/smart-home-for-avs.html)または次の[ブログ記事](https://developer.amazon.com/en-US/blogs/alexa/device-makers/2020/04/create-a-sample-alexa-built-in-disco-ball-with-smart-home-for-av)を参照してください。

利用可能な他のインターフェイスについて詳しく知りたい場合、または AWS IoT EduKit リファレンスハードウェア用 M5Stack Core2 の他の機能を使って独自の実装を考案する場合は、[ModeController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-modecontroller.html)および [ToggleController](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/alexa-togglecontroller.html)をご覧ください。創造力とこのチュートリアルの他の章で学んだことを活用して、音声の新しい活用方法を探求しましょう。

これは、現在利用可能な最後のチュートリアルです。今後さらにパブリッシュする予定ですが、現時点で、ユーザーは、これまでに学んだスキルを活用しながら、AWS サービスを利用して独自の IoT ソリューションを構築するためのスキルを身につけています。

## クリーンアップ
このソリューションでは、AWS でユーザー自身のリソースを使用しませんでした。しかし、Alexaと連携するこのチュートリアルを進めていた場合は、以下のコマンドをPlatformIO CLI[ターミナルウィンドウ](/jp/blinky-hello-world/prerequisites.html#platformio)に入力することでフラッシュの削除が行なえます(証明書もフラッシュから削除されます)

```bash
pio run --environment core2foraws --target erase
```

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}