+++
title = "機械学習モデルの使用"
weight = 40
pre = "<b>d. </b>"
+++

## 本章の概要
この章の終わりには、サーバーレスアプリケーションで次のようなことができるようになります。

* 新しいサーモスタットメッセージを *roomOccupancy* の推論に変換する機械学習モデルを使用する

## サーバーレスインフラストラクチャのセットアップ方法
次のステップでは、AWS Lambda でサーバーレス関数を作成する方法を詳しく説明しています。関数は IoT Core からの Device Shadow メッセージを待ち、メッセージを機械学習エンドポイントで使われる形式に変換し、その後機械学習エンドポイントを呼び出して *roomOccupancy* の分類と推論の信頼スコアを返すコードの一部を定義します。

1. [AWS Lambda コンソール](https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#)から、[Create function (関数を作成)] を選択します。
2. 関数に名前を付けます。今後のステップでは `classifyRoomOccupancy` という名前を使用します。
3. [ランタイム] の下にある [Python 3.8] を選択します。
4. [Create function (関数を作成)] を選択します。
5. [関数コード] の下の lambda_function.py ファイルに、次のコードをコピーして貼り付け、プレースホルダーコードを置き換えます。

```python
import json
import boto3
import os

# Receives a device shadow Accepted document from IoT Core rules engine.
# Event has signature like {"state": {"reported": {"sound": 5}}}.
# See expectedAttributes for full list of attributes expected in state.reported.
# Builds CSV input to send to SageMaker endpoint, name of which stored in
#   environment variable SAGEMAKER_ENDPOINT.
#
# Returns the prediction and confidence score from the ML model endpoint.
def lambda_handler(event, context):
    client = boto3.client('sagemaker-runtime')
    
    print('event received: {}'.format(event))
    
    # Order of attributes must match order expected by ML model endpoint. E.g.
    #   the same order of columns used to train the model.
    expectedAttributes = ['sound', 'temperature', 'hvacStatus', 'roomOccupancy', 'timestamp']
    reported = event['state']['reported']
    reported['timestamp'] = event['timestamp']
    reportedAttributes = reported.keys()
    
    # Validates the input event has all the expected attributes.
    if(len(set(expectedAttributes) & set(reportedAttributes)) < len(expectedAttributes)):
        return {
            'statusCode': 400,
            'body': 'Error: missing attributes from event. Expected: {}. Received: {}.'.format(','.join(expectedAttributes), ','.join(reportedAttributes))
        }
    
    # Build the input CSV string to send to the ML model endpoint.
    reportedValues = []
    for attr in expectedAttributes:
        reportedValues.append(str(reported[attr]))
    input = ','.join(reportedValues)
    print('sending this input for inference: {}'.format(input))

    endpoint_name = os.environ['SAGEMAKER_ENDPOINT']
    content_type = "text/csv"
    accept = "application/json"
    payload = input
    response = client.invoke_endpoint(
        EndpointName=endpoint_name, 
        ContentType=content_type,
        Accept=accept,
        Body=payload
        )
        
    body = response['Body'].read()
        
    print('received this response from inference endpoint: {}'.format(body))
    
    return {
        'statusCode': 200,
        'body': json.loads(body)['predictions'][0]
    }
```

6. [環境変数] の下にある [編集] を選択し、開いた画面で[環境変数の追加]を選択します。
1. [キー] には `SAGEMAKER_ENDPOINT` と入力し、[値] には SageMaker エンドポイント名を入力します。前章の最後のステップでこのリソースに名前を付けました。このモジュールでは `roomOccupancyEndpoint` という名前とします。
1. [保存] を選択して、この新しい環境変数をコミットし、メインの Lambda エディタインターフェイスに戻ります。
1. [Designer (デザイナー)] パネルで、[トリガーを追加] を選択します。
1. [Trigger configuration (トリガーの設定)] では、[AWS IoT] をリストから選択します。
1. [IoT type (IoT タイプ)] では、[Custom IoT rule (カスタム IoT ルール)] を選択します。
1. [ルール] では、サーモスタットからの Device Shadow メッセージを処理し、roomOccupancy 値の新しいメッセージを発行するルールをリストから見つけます。前のモジュールの「スマートサーモスタット」で、このルールに `thermostatRule` という名前を付けています。
1. [Enable trigger (トリガーの有効化)] チェックボックスが有効になっていることを確認し、[追加] を選択します。これは IoT Core ルールに対して、この Lambda 関数を呼び出す許可を与えます。
1. [アクセス権限] タブを選択後、実行ロールの[ロール名] の下にあるリンクを選択します。
1. IAM コンソールで開いた新しいタブから、[Permissions policies] の下の [Add inline policy (インラインポリシーを追加)] を選択します。
1. [サービス] では、[SageMaker] を選択します。
1. [アクション] では、[InvokeEndpoint] を選択します。
1. [リソース] では、[すべてのリソース] を選択します。
1. [ポリシーの確認] を選択します。
1. ポリシーに `invokeSageMakerEndpoint` のような名前を付け、[ポリシーの作成] を選択します。ブラウザこのIAMコンソールのタブを閉じても構いません。

これらのステップでもって、AWS Lambda 関数の設定が完了します。例えば、Lambda 関数が次の Device Shadow の更新を受信した場合、

```JSON
{
    "state": {
        "reported": {
            "sound": 20,
            "temperature": 58.8,
            "hvacStatus": "HEATING",
            "roomOccupancy": true
        }
    },
    "timestamp": 1234567890
}
```

SageMaker エンドポイントの呼び出し後、次のレスポンスを返します。

```JSON
{
  "statusCode": 200,
  "body": {
    "predicted_label": "false",
    "probability": "0.9999991655349731"
  }
}
```

次のステップでは、Lambda 関数統合を使用するため、IoT Core ルール (`thermostatRule` という名前のルール) を更新します。

1. IoT Core コンソールに戻り、[Act]、[ルール] を選択し、`thermostatRule`をクリックします。
2. [Rule query statement (ルールクエリステートメント)] の近くにある [編集] を選択します。現在 `SELECT CASE state.reported.sound > 10 WHEN true THEN true ELSE false END AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null` を読み込んでいます。
3. このクエリを次の新しいものに置き換えます。

```SQL
SELECT cast(get(get(aws_lambda("arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME", *), "body"), "predicted_label") AS Boolean) AS state.desired.roomOccupancy FROM '$aws/things/<<CLIENT_ID>>/shadow/update/accepted' WHERE state.reported.sound <> Null
```

4. プレースホルダーの `REGION` (コンソールヘッダーで現在のリージョンを確認します。`Oregon` ではなく `us-west-2` のような形式になっています)、`ACCOUNT_ID` (ユーザー名が表示されたコンソールヘッダーメニューに、ハイフンのない 12 桁の数字として記載されています)、`FUNCTION_NAME` (作成した AWS Lambda 関数名で `classifyRoomOccupancy` という名前になっています) を必ず置き換えてください。FROM トピックの **<<CLIENT_ID>>** プレースホルダーも忘れずに変更しますしてください。
1. [アクション] の下にある [Send a message to a Lambda function (Lambda 関数にメッセージを送信)] という名前のアクションを見つけ、[削除] を選択します。これはこの IoT Core ルールが Lambda 関数 を呼び出せるよう、Lambda 関数設定でトリガーを作成した際にデフォルトで追加されたものですが、アクションは必要ありません。代わりに、ルールクエリのインライン呼び出しを使っていますが、トリガーが追加したものと同じアクセス許可が必要でした。
1. [更新] を選択します。

これで、IoT ワークフローはデプロイされたエンドポイントからトレーニング済み機械学習モデルを使用して、スマートサーモスタットが発行したメッセージを新しい **roomOccupancy** 値として分類するようになります。

## 検証ステップ
次章に進む前に、サーバーレスアプリケーションが想定どおりに設定されているかを検証できます。

1. AWS IoT Core Test クライアントを使用して、トピック `$aws/things/<<CLIENT_ID>>/shadow/update` (**<<CLIENT_ID>>** は置換) にサブスクライブすると、こちらに 2 種類のメッセージが表示されます。1 つ目はスマートサーモスタットが発行した `state.reported` パスのペイロードです。もう 1 つは、サーモスタットルールにより発行されるようになった、機械学習モデルが決める `state.desired.roomOccupancy` 値のペイロードです。

想定どおりに機能していれば、このモジュールは完了です。[まとめ](/jp/smart-spaces/conclusion.html)に進んでください。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}