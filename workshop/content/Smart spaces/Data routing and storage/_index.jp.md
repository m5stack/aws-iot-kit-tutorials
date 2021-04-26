+++
title = "データのルーティングとストレージ"
weight = 20
pre = "<b>b. </b>"
+++

## 本章の概要
この章の終わりには、サーバーレスアプリケーションで次のようなことができるようになります。

* スマートサーモスタットから受信したメッセージをマネージドストレージおよび分析サービスに転送する
* 処理済みデータに対してクエリを実行し、マテリアライズドビューの結果を生成できる

## IoT データの保存および分析の概念
この時点までは、IoT ソリューションのあらゆる側面は、各メッセージの受信、処理、その後の廃棄という意味では一時的なものとなっています。IoT Events 探知器モデルの場合、新規メッセージに対応するステートフルエンティティはありますが、それ以外ではサーモスタットメッセージの履歴は保存されていません。このステップでは、サーモスタットメッセージのデータストアを構築します。

AWS にはクラウドにデータを保存するさまざまな方法があり、IoT データの保存も同様です。このソリューションでは [AWS IoT Analytics](https://aws.amazon.com/iot-analytics/) というサービスの使用を推奨しています。これは IoT データの受信、保存、処理、分析を一括で行う専用のマネージドサービスです。IoT Analytics を使用して、Device Shadow への更新のそれぞれのサブセットを保存します。

AWS IoT Analyticsのドキュメントでは、これについて詳細を網羅していますが、そのしくみの概要は次のとおりです。サービスのエントリポイントは、チャネルと呼ばれるリソースです。チャネルはワークフローのすべての未加工データを保存します。また、受信した各メッセージのコピーを、パイプラインと呼ばれる次のリソースに送信します。パイプラインは一連のアクティビティで、データが分析ユースケースで使用される前に、データの処理、クレンジング、フィルター、エンリッチを行います。処理されたメッセージはパイプラインからデータストアに入ります。データストアはチャネルと同様に、処理データの永続的なストレージユニットです。最後にデータセットが AWS IoT Analytics プロジェクトの最後のリソースです。データセットは SQL に似たクエリを定義します。このクエリはデータストアからマテリアルズド・ビューとしてメッセージを読み取り、クエリのコンテンツを S3 バケットといった送信先に配信します。

スマートスペースソリューションは、サーモスタットからの数時間のランタイムデータを AWS IoT Analytics に蓄積します。その後、データセットクエリの結果によって、このデータが Amazon SageMaker の機械学習ツールチェーンで使用可能になります。

[AWS IoT Analytics](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html)にはより多くの機能がありますが、このモジュールの目的からは、この方法が最も簡単にサーモスタットメッセージの履歴を保存し、機械学習モデルのトレーニングデータセットとして蓄積できる方法です。

{{% notice warning %}}
このアプリケーションを6時間以上実行したままにすると、AWSの料金が発生する可能性があります。不必要なコストが懸念される場合は、チュートリアルを終了したら、すぐに[削除](/jp/smart-spaces/conclusion.html#heading-1)することをおすすめします。
{{% /notice %}}

## サーバーレスインフラストラクチャのセットアップ方法
次のステップでは、新しい IoT Core ルール、新しい AWS IoT Analytics プロジェクトを作成する方法、ルールを使用して Device Shadow メッセージを AWS IoT Analytics プロジェクトに転送する方法を詳しく説明します。IoT Core ルール作成ウィザードには便利なインターフェイスがあり、AWS IoT Analytics プロジェクト全体をユーザーに代わって作成します。

1. [AWS IoT Core コンソール] で、[Act]、[ルール]、[作成] の順に選択します。
2. ルールに **storeInIoTAnalytics** のような名前を付け、説明を記入します。
3. 次のクエリを使用します。必ず **<<CLIENT_ID>>** を Core2 for AWS IoT Edukit リファレンスハードウェアキットの画面に表示されているクライアント ID/シリアルナンバーに置き換えてください。

```SQL
SELECT current.state.reported.sound, current.state.reported.temperature, current.state.reported.hvacStatus, current.state.reported.roomOccupancy, timestamp FROM '$aws/things/<<CLIENT_ID>>/shadow/update/documents'
```

4. [アクションの追加]、[Set one or more actions (1 つ以上のアクションを設定)] を選択します。
5. [Send a message to IoT Analytics (IoT Analytics にメッセージを送信)]、[アクションの設定] の順に選択します。
6. [Quick create IoT Analytics resources (IoT Analyticsリソース をすばやく作成する)] を選択し、[Resource prefix (リソースプレフィックス)] にプロジェクト名を入力します。モジュールの今後のステップでは、プレフィックスに **smartspace** を使用します。[Quick Create (クイック作成)] を選択すると、すべての AWS IoT Analytics リソースが作成され、自動的に設定されます。
7. [アクションの追加] を選択して、このアクションの設定を終了し、ルール作成フォームに戻ります。
8. [Create rule (ルールの作成)] を選択し、新規ルールの作成を終了します。

## 検証ステップ
次章に進む前に、サーバーレスアプリケーションが想定どおりに設定されているかを検証できます。

1. スマートサーモスタットの電源が入っていてデータを発行しており、トレーニングを行う部屋に設置されていることを確認します
1. [AWS IoT Analytics コンソール](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets)を使用して、最新のデータセットの中身を確認し、周囲の雑音レベル、気温、部屋の在室状況の履歴データが入っているかを検証します。この確認を行うには、[IoT Analytics コンソール](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets) でプロジェクト名で指定したデータセットの詳細画面を開き、[アクション]、[Run now (すぐに実行)] の順に選択し、[Result preview (結果プレビュー)] が最新の内容に更新されるまで待ちます。次のような結果が表示されます

{{< img "iot_analytics-dataset_run.en.png" "Running the data set" >}}
{{< img "iot_analytics-dataset_preview.en.png" "Preview of the data set" >}}

想定どおりに機能している場合は、[機械学習](/jp/smart-spaces/machine-learning.html)に進みましょう。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}