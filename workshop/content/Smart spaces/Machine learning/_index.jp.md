+++
title = "機械学習"
weight = 30
pre = "<b>c. </b>"
+++

## 本章の概要
この章の終わりには、サーバーレスアプリケーションで次のようなことができるようになります。

* 新しいサーモスタットメッセージを roomOccupancy の推論に変換する機械学習モデルをトレーニングする
* 機械学習モデルを API エンドポイントにホストし、使用する

## 機械学習モデルのトレーニングの概念
データサイエンスと機械学習は、それ自体が非常に大規模なドメインです。機械学習モデルのトレーニングのしくみの基本を説明することは、このモジュールの範囲をはるかに超えています。幸いにも、新しいモデルを作成するツールチェーンは、データについて十分理解していれば、ツールチェーンを使用して機械学習の実験が可能なほど簡略化されています。

このソリューションでは、IoT Core ルールエンジンの簡易しきい値を置き換えます。簡易しきい値はデバイスが通知する音声レベルを評価し、*roomOccupancy* と呼ばれる新しいブールキー値に強制的に変換します。通知されたメッセージの音声データを見ると、静かなときは値が低く、雑音があるときは高くなることがわかります。このことから、「10 より大きい」というような簡易しきい値は *roomOccupancy* 値を生成する出発点として適切であることがわかります (特定のケースでは、登録済みのアクティビティに対して周囲の雑音の異なるしきい値を設定する方がより適切な場合もあります)。

同様のアプローチで、機械学習ツールチェーンに、デバイスが設置されている部屋で記録されたサーモスタットデータのサンプルを提供し、「roomOccupancy が真のときデータはこのようになり、偽のときはこのようになる」というトレーニングジョブを伝えます。 トレーニングジョブはデータセットを評価し、*roomOccupancy* 列を対象にして、音声レベル範囲、さらには時間、HVAC 状態、温度といったその他の列に基づき、*roomOccupancy* 値を正確に再生成するモデルをビルドしようとします。

最初のモデルのトレーニングが完了すると、次の章で、音声レベルの簡易静的しきい値をモデルが決定した *roomOccupancy* の推論分類に置き換えます。

## サーバーレスインフラストラクチャのセットアップ方法
自動モデルトレーニングの新たな実験の設定を行うためには、まず Amazon SageMaker Studio をセットアップする必要があります。

1. [Amazon SageMaker コンソール](https://us-west-2.console.aws.amazon.com/sagemaker/home?region=us-west-2#/dashboard)に移動し、[Amazon SageMaker Studio] を選択します。
1. [クイックスタート] を選択し、任意で新しい [ユーザー名] を入力します。
1. [実行ロール] では、ドロップダウンから [新しい IAM ロールの作成] を選択します。
1. [S3 buckets you specify (指定する S3 バケット)] では、[なし] を選択し、[ロールの作成] をクリックします。「sagemaker」と名前が付いたバケットのその他のデフォルトは、このプロジェクトの場合そのままで構いません。
1. [送信] を選択し、SageMaker Studio のプロビジョニングプロセスを開始します。このステップには数分かかり、ユーザーに代わって完了します。

SageMaker Studio のプロビジョニングが完了(完了まで少し時間がかかります)すると、次のステップでは Studio を開き、新しいプロジェクトを設定します。

1. SageMaker Studio コントロールパネルからユーザー名を選択し、[Open Studio (Studio を開く)] を選択します。
1. [ローンチャー] タブで、[New project (新規プロジェクト)] を選択します。
1. [SageMaker project templates (SageMaker プロジェクトテンプレート)] の下にある [MLOps template for model building, training, and deployment (モデルビルド、トレーニング、デプロイ用 MLOps テンプレート)]を選択状態にし、[Select project template (プロジェクトテンプレートを選択)] を選択します。
1. プロジェクトに名前を付けて説明を記入し、[Create project (プロジェクトを作成)] を選択します(完了まで少し時間がかかります)。

プロジェクトが作成されると、[Repositories]、[Pipelines]、[Experiments] などのタブがあるプロジェクトダッシュボードが表示されます。このブラウザのタブは SageMaker Studio で開いたままにして、このページにすばやく戻れるようにしておきます。

次のタスクでは AWS IoT Analytics に戻り、新しい機械学習プロジェクトで使用するために、蓄積したサーモスタットデータをエクスポートします。

1. [[AWS IoT Analytics console](https://us-west-2.console.aws.amazon.com/iotanalytics/home?region=us-west-2#/datasets)] を開き、データセットを選択します (`smartspace_dataset` という名前になっています)。
2. [Data set content delivery rules (データセットコンテンツ配信ルール)] で、[編集] を選択します。
3. [ルールを追加]、[Deliver result to S3 (S3 に結果を配信)] の順に選択します。
4. [S3 バケット] で、[Please select a resource (リソースを選択してください)] を選択し、SageMaker Studio プロジェクト用に作成された S3 バケットを見つけます。`sagemaker-project-p-somehashhere` というような名前が付いています。このような名前のバケットが複数ある場合には、SageMaker Studio プロジェクトでプロジェクトのランダムハッシュ ID を確認する必要があります。[リポジトリ] や [パイプライン] タブなどのプロジェクトの他のリソースで、ハッシュを確認できます。
5. [Bucket key expression (バケットキー表現)] で、`data/smartspace/Version/!{iotanalytics:scheduleTime}_!{iotanalytics:versionId}.csv` という式を使用します。
6. [ロール] で [Create new (新規作成)] を選択し、S3 バケットにデータを書き込む IoT Analytics アクセスを付与する IAM ロールの名前を付けます。[ロールの作成] を選択します。
7. [保存] を選択し、新規配信ルールを終了します。

{{< img "iot_analytics-dataset_delivery.en.png" "Content delivery rule" >}}

1. 新しい Amazon S3 バケットにトレーニング用として保存されるデータセットを生成するには、[アクション]、[Run now (今すぐ実行)] の順に選択します。データセットコンテンツの生成完了後、[Result preview (結果プレビュー)] の更新を確認する必要があります。

{{< img "iot_analytics-dataset_run.en.png" "Running the data set" >}}
{{< img "iot_analytics-dataset_preview.en.png" "Preview of the data set" >}}

これで、SageMaker Studio に戻り、機械学習実験を開始する準備ができました。実験では、通知されたサーモスタットデータを使用します。これは IoT Analytics データセットより入力として今エクスポートされたものです。実験を設定して、既存の roomOccupancy 列を正確に予測する方法を見つけ出します。自動トレーニングジョブは関連するアルゴリズムの試行のため、データを分析します。その後、250 のトレーニングジョブを異なるハイパーパラメータで実行し、入力トレーニングデータに最も適切なものを選択します。

機械学習の実験を開始する前に、分析する部屋のサーモスタットから通知された数時間のデータを保持している必要があります。また、その数時間の間、部屋にはアクティブな時間とアクティブではない時間が混在している必要があります。自動機械学習実験で有効な結果を得るにはは、少なくとも 500 行のデータが必要です。ただし、より多くのデータがあれば、さらによい結果が得られます。先に進む前にまだ多くのデータを生成する必要がある場合は、IoT Analytics コンソールのデータセットを再実行 (先ほどの手順リストの最後のステップ) してみてください。これらの結果がプロジェクトの S3 バケットの SageMaker で利用可能になります。実験を開始する準備ができたら、以下を読み進めてください。

1. SageMaker Studio に戻り、プロジェクトを開いて、[Experiments] タブ、[Create autopilot experiment] の順に選択します。
2. [Experiments name]を付けます。
3. [Project] で、リストからプロジェクトを選択します。
4. [Connect your data]、[S3 bucket name] で、リストからプロジェクトの S3 バケットを選択します。前章の IoT Analytics データセットのコンテンツ配信ルールで選択したものと同じものです。
5. [Dataset file name] で、`data/smartspace/Version/1607276270943_3b4eb6bb-8533-4ac0-b8fd-1b62ac0020a2.csv` のような IoT Analytics データセットコンテンツを見つけ、選択します。
6. [Target] で、`roomoccupancy` を選択します。
7. [Output data location]、[S3 bucket name] で、ステップ 4 で選択したプロジェクトの S3 バケットと同じものをこのリストから見つけ、選択します。
8. [Dataset directory name] に `output/smartspace` と入力し、[Use input as S3 object key prefix “output/smartspace”] を選択します。これにより、S3 バケットで新しいプレフィックスが定義され、出力ファイルで使用されるようになります。
9. [Create Experiment] を選択し、自動機械学習実験を開始します。

実験の実行には数時間かかります。こちらの SageMaker Studio ブラウザタブで、実験の進捗を確認できますが、タブを閉じてまた後で開いて、進捗を確認することもできます。

実験完了後に結果として得られる出力は 250 の試験で、SageMaker が最適なチューニングのジョブパラメータを見つけるために使用したものです(表示されない場合はブラウザの再表示を実行してください)。試験結果の表を並び替え、[BEST (最適)] とマークされたものを見つけます。次のマイルストーンでは、この試験をモデルエンドポイントとしてデプロイし、API としてそれを呼び出せるようにします。

1. [Best] とマークされた試験を選択し、[Deploy Model] を選択します
  {{< img "sagemaker-trials.png" "SageMaker trials" >}}
2. エンドポイントに`roomOccupancyEndpoint`と名前を付けます([REALTIME DEPLOYMENT SETTINGS]が閉じている場合は開きます)。
3. [Inference Response Content (レスポンスコンテンツの推論)] で、[predicted_label] と [probability] の両方を選択します。[predicted_label] はリストにすでに追加されている場合があります。
4. [Deploy model] を選択し、SageMaker にモデルを新しい利用可能な API エンドポイントとしてデプロイするよう指示します。これには数分かかります。

{{< img "sagemaker-deploy.png" "SageMaker deploy" >}}

これで、機械学習モデルが API エンドポイントとしてデプロイされ、Amazon SageMaker で管理されるようになりました。次章の「機械学習モデルの使用」では、API エンドポイントをサーバーレス関数で使用し、IoT Core ルールの簡易しきい値ロジックを置き換えます。これにより、モデルが生成した推論で `roomOccupancy` 値を決定します。

{{% notice warning %}}
このアプリケーションを6時間以上実行したままにすると、AWSの料金(S3へのリクエスト)が発生する可能性があります。不必要なコストが懸念される場合は、チュートリアルを終了したら、すぐに[削除](/jp/smart-spaces/conclusion.html#heading-1)することをおすすめします。
{{% /notice %}}

## 検証ステップ
次章に進む前に、サーバーレスアプリケーションが想定どおりに設定されているかを検証できます。

1. [Amazon SageMaker コンソール](https://us-west-2.console.aws.amazon.com/sagemaker/home?region=us-west-2#/endpoints) を使用すると、[エンドポイント] ページで、新しいエンドポイントの状態が [InService] となっていることを確認できます。

{{< img "sagemaker-endpoints.png" "SageMaker endpoints" >}}

想定どおりに機能している場合は、[機械学習モデルの使用](/jp/smart-spaces/working-with-ml-models.html)に進みましょう。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}