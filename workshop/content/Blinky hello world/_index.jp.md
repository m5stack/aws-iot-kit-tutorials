
+++
title = "Lチカ"
chapter = true
weight = 30
pre = "<b>2. </b>"
+++

AWS IoT EduKit 用 M5Stack Core2 リファレンスハードウェアを使用して、「Lチカ」 (マイクロコントローラーの Hello World ) アプリケーションを作成する方法を学びます。ユーザー自身の AWS アカウントを使って、AWS IoT Core に接続し、メッセージを送受信し、デバイスと AWS クラウドの両方でメッセージを表示する方法を学びます。ここで使用されているすべての手順とスキルは、この後に続くチュートリアルで成功するための基礎となります。具体的には、以下を行います。

- 追加の必要なソフトウェアのインストールし、設定します
- オンボードのセキュアエレメントに、事前に用意されているセキュリティ証明書を使用して、AWS IoT Core に「モノ」を登録します。
- ESP-IDF (FreeRTOS kernel with symmetric multiprocessing) を使用して、リファレンスハードウェアから AWS IoT Core に MQTT メッセージを接続して送信します
- リファレンスで MQTT メッセージを受信し、LEDを点滅させます

このチュートリアルのすべてのコンテンツは、ユーザーが、[M5Stack Core2 ESP32 IoT Development Kit for AWS](https://ssci.to/Core2_for_AWS)を所有し、本番環境で利用していない[AWS アカウント](https://console.aws.amazon.com/console/home)を持っていること、そして、コマンドプロンプト/ターミナルなどの基本的な技術概念やツールに慣れていることを前提としています。ユーザー自身のキットを購入するには、[スイッチサイエンス](https://ssci.to/Core2_for_AWS)のオンラインショップもしくは、[M5Stack ストア](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) をチェックしてください。AWS アカウントにまだ登録していない場合は、[アカウントを作成](https://portal.aws.amazon.com/billing/signup)してください。

このチュートリアルを開始するには、最初の章[前提条件](/jp/blinky-hello-world/prerequisites.html)に進んでください。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}