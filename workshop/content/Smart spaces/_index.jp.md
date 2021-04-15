+++
title = "スマートスペース"
chapter = true
weight = 50
pre = "<b>4. </b>"
+++

このチュートリアルでは、前のモジュール「スマートサーモスタット」ソリューションを「スマートスペースソリューション」に拡張します。スマートスペースとは、スペースに関するインサイトを使ってスペースを強化する、またはスペースにさらなる機能をもたらすという概念です。アナリティクスと機械学習機能を使って、スマートサーモスタットが設置されている部屋の在室状況に関する生データから予測を導き出します。スマートスペースソリューションでは、サーモスタットのデータから新しい機械学習モデルを作成する方法、および部屋の在室状況の分類を改善してスマートサーモスタットをより一層適切に運用する方法を紹介します。

このラボを始める前に、以下の前提条件を確認してください。

* AWS IoT EduKit 用の M5Stack Core2 ESP32 IoT 開発キットを持っている。
* プロダクションワークロードを実行していない AWS アカウント (すなわちサンドボックスおよび評価目的用の安全なアカウント) を持っている。
* 管理者アクセス権限を持つ AWS アカウントのユーザーログイン、またはロールを持っている。
* Core2 for AWS IoT EduKit は AWS IoT Core でプロビジョニングされており、MQTT を介してすでに AWS と通信している。その機能のベースラインがまだ完了していない場合は、[スマートサーモスタット](/jp/smart-thermostat.html)のチュートリアルから始めてください。


学習の目標：このラボを終えると、以下を習得できます。

1. デバイスのテレメトリを AWS IoT Analytics に転送し、ストレージと変換を行い、分析データセットを作成する方法。
1. Amazon SageMaker Studio を使用して、データセットから機械学習モデルを自動生成する実験を作成する方法。
1. 機械学習モデルを消費可能な API エンドポイントにデプロイする方法。
1. サーバーレス関数がどのようにして機械学習の推論 API を消費してアプリケーションを拡張しているか。

まずは[はじめに](/jp/smart-spaces/introduction.html)から開始しましょう。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}