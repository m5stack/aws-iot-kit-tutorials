+++
title = "スマートサーモスタット"
chapter = true
weight = 40
pre = "<b>3. </b>"
+++

このチュートリアルでは、リファレンスハードウェアを架空の HVAC(Heating Ventilation and Air Conditioning) システムを制御するスマートサーモスタットに構成します。スマートサーモスタットは、測定された室温と騒音レベルをクラウドの Device Shadow に通知します。また、ユーザーは、通知された測定値をリッスンしてサーモスタットが設定すべき状態を判断し、何をすべきかを指示するコマンドをデバイスに送り返すサーバーレスアプリケーションを構成します。

前提このチュートリアルを始める前に、以下の前提条件を確認してください。
1. AWS IoT EduKit 用の [M5Stack Core2 ESP32 IoT 開発キット](https://ssci.to/Core2_for_AWS)を持っている
2. プロダクションワークロードを実行していない AWS アカウント (すなわちサンドボックスおよび評価目的用の安全なアカウント) を持っている
3. 管理者アクセス権限を持つ AWS アカウントのユーザーログイン、またはロールを持っている
4. Core2 for AWS IoT EduKit は AWS IoT Core でプロビジョニングされており、MQTT を介してすでに AWS と通信している。プロビジョニングを完了して接続を確立していない場合は、[Lチカ](/jp/blinky-hello-world.html) チュートリアルから始めてください
5. 少なくとも、トピック、パブリッシュ、サブスクライブなどの [AWS IoT メッセージング](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html)の基本的な技術概念について理解している


学習の目標。このラボを終えると、以下を習得できます
1. AWS IoT EduKit デバイス用の Core2 から温度と音のレベルを取得する方法
2. 温度と音の測定値を、デバイスから AWS IoT Core にパブリッシュする方法
3. 測定値を Device Shadow に通知する方法
4. AWS IoT Core ルールエンジンを使用してメッセージ変換を実行する方法
5. 入力に応答し複雑なイベントを検出するサーバーレスアプリケーションを構築する方法
6. Device Shadow を介してデバイスにコマンドを送信する方法

このチュートリアルを開始するには、最初の章[はじめに](/jp/smart-thermostat/introduction.html)に進んでください。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}