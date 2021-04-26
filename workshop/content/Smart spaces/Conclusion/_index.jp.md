+++
title = "まとめ"
weight = 50
pre = "<b>e. </b>"
+++

## まとめ
スマートサーモスタットアプリケーションをアップグレードしてトレーニング済み機械学習モデルを活用する、実践的モジュールが完了しました。これで IoT アプリケーション構築のエンドツーエンドエクスペリエンスを一通り確認しました。それには、センサーからの物理データの取得、エッジとクラウド間の状態の同期、エッジデバイスをコントロールするサーバーレスアプリケーションの構築、行動につながるインサイトへの機械学習によるデータの変換が含まれます。

引き続きソリューションの拡張を行う場合の例としては、次のものがあります。

* IoT Analytics のデータセットの作成スケジュールを設定し、最新のコンテンツファイルを 1 日 1 回作成する
* 簡易なウェブ UI を構築するか、Amazon QuickSight をプロビジョニングして、IoT Analytics で作成されたデータセットを可視化する
* 新しいパイプラインアクティビティを IoT Analytics に追加して、Lambda 関数経由で局地天気を取得し、サーモスタットデータに追加する
* Core2 for AWS IoT EduKit リファレンスハードウェアのライブラリを使って、*hvacStatus* が変化した場合は常に音を発するようにする
* Core2 for AWS IoT EduKit リファレンスハードウェアのライブラリを使って、*roomOccupancy* が変化した場合は常に LED を点滅するようにする

## クリーンアップ
このソリューションと以前のソリューション 「スマートサーモスタット」 を通じて、以下のリソースを AWS に作成しました。

* IoT Core ルール
* IAM ロール
* IoT Events 探知器モデル
* IoT Analytics プロジェクト (チャネル、パイプライン、データストア、データセット)
* Lambda 関数
* SageMaker Studio
* S3 バケット (SageMaker Studio プロジェクトの一部)
* SageMaker モデルグループ
* SageMaker エンドポイント

アカウントでは以下の 3 種類の方法の従量制課金が継続して発生します。まず、SageMaker エンドポイントのようなプロビジョニングされたコンピューティングリソースで、仮想インスタンスにホストされている場合。これらのリソースは時間単位で課金されます。次に、S3 バケット、IoT Analytics プロジェクト (チャネル、データストア、処理済みデータセット結果) のようなリソースのデータストレージの場合。これらのリソースはギガバイト/月のような請求ディメンションとなります。最後に、IoT Core へのメッセージの発行、Lambda 関数の呼び出し、IoT Events でのイベントの処理のようなイベント駆動型アクティビティの場合は、イベントごとに課金されます。デバイスの電源をオフにした場合は、イベント駆動型アクティビティは中止されます。ソリューションの使用が終了し、これらの AWS リソースを引き続き使用する予定がない場合は、これらのリソースを破棄する必要があります。

SageMaker エンドポイントは時間単位の課金となるため、最もコストがかかるリソースです。このリソースの使用が終わった場合には、破棄することをお勧めします。モデルそのものは保持され、後で再度デプロイできます。SageMaker エンドポイント破棄のステップ:

1. SageMaker マネジメントコンソールに移動し、[エンドポイント] を選択します。リストからエンドポイントを見つけ、削除します。
2. SageMaker Studioの削除は[こちら](https://docs.aws.amazon.com/sagemaker/latest/dg/gs-studio-delete-domain.html#gs-studio-delete-domain-studio)を参考にしてください

IoT Analytics ストレージリソース破棄のステップ:

1. AWS IoT Analytics に移動し、[チャネル] を選択します。リストからチャネルを見つけ、削除します。
2. AWS IoT Analytics に移動し、[データストア] を選択します。リストからデータストアを見つけ、削除します。
3. AWS IoT Analytics に移動し、[データセット] を選択します。リストからデータセットを見つけ、削除します。

IoT Core ルール、Lambda 関数、IoT Events 入力および検知、IoT Analytics パイプラインは、使用した場合に料金が発生します。今後それらが必要ない場合は、それぞれのマネジメントコンソールのリソース詳細ページから削除することによって破棄できます。

1. AWS IoT Core に移動し、[Act]、[Rules (ルール)] の順に選択し、リストからルールを見つけ、削除します。
2. AWS IoT Analytics に移動し、[パイプライン] を選択します。リストからパイプラインを見つけ、削除します。
3. AWS Lambda に移動し、[関数] を選択します。リストから関数を見つけ、削除します。
4. AWS IoT Events に移動し、[入力] を選択します。リストから入力を見つけ、削除します。
5. AWS IoT Events に移動して、[Detector models (探知器モデル)] を選択し、リストからモデルを見つけて削除します。

次のチュートリアルである [Alexa for IoT 入門](/jp/intro-to-alexa-for-iot.html)に移動します。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}