+++
title = "Alexa for IoT 入門"
chapter = true
weight = 60
pre = "<b>5. </b>"
+++

このチュートリアルでは、AWS IoT EduKit リファレンスハードウェアキット用 M5Stack Core2 に、AWS IoT (AFI) 用 AlexaVoice Service Integration を実装します。Espressif Voice Assistant SDK (VA-SDK) とAlexa を使用して、Alexa スマートホームコマンドを使いながらオンボード LED を制御する方法を学習します。このチュートリアルでは現在、Espressifが提供するAWSアカウントを使用しています。これは、AWS IoT EduKit リファレンスハードウェア向けの、M5Stack Core2 用 Espressif VA-SDK の`ベータポート`です。

前提このチュートリアルを始める前に、以下の前提条件を確認してください。

1. [M5Stack Core2 ESP32 IoT Development Kit for AWS IoT EduKit](https://ssci.to/Core2_for_AWS)を持っている。
2. [**開始方法**](/jp/getting-started.html) と [**Lチカ**](/jp/blinky-hello-world.html) チュートリアルを完了し、開発環境を設定してファームウェアをコンパイルすることができ、デバイスにファームウェアをアップロードすることができる。
3. 少なくとも、トピック、パブリッシュ、サブスクライブなどの AWS IoT メッセージングの基本的な技術概念について理解している。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}