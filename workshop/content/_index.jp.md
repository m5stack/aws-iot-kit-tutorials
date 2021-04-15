+++
title = "AWS IoT EduKit"
chapter = true
weight = 1
+++
![EduKit Logo](AWS_IoT_EduKIt_Logo-320px_193px.png)
AWS IoT EduKit では、AWS サービスを利用してIoT アプリケーションを構築する方法を、ワークショップを通して簡単に学ぶことができます。AWS IoT EduKit では、リファレンスハードウェアキットとわかりやすい一連のガイドとサンプルコードで構成されており、学生から経験豊富なエンジニアやプロフェッショナルに至るデベロッパーが、エンドツーエンドの IoT アプリケーションを構築する実践的な経験を得るお手伝いをします。

### AWS IoT EduKit のメリット
#### ハードウェア選択の簡略化

- AWS IoT EduKit のリファレンスハードウェアキットは、当社の製造パートナーである M5Stack 社が製造・販売を行い、多数の IoT アプリケーションを箱から出してすぐに使えるようにするオンボード機能が数多く詰まっています。デベロッパーは、プラグアンドプレイ拡張オプションを使って、追加のユースケースにも対応できるよう機能を拡張することができます。
- リファレンスハードウェアキットは、さまざまな IoT アプリケーション (エントリレベルから上級/プロフェッショナルレベルまで) を構築するための強力かつ安全なハードウェアを、エントリレベルの価格で提供します。リファレンスハードウェアキットは、Microchip 社のATECC608 Trust&GO セキュアエレメントが結合された Espressif ESP32 MCU を搭載しています。また、Alexa に対応しており、複数のオンボード周辺機器、追加接続オプション (LoRaWAN、NB-IoT、LTS など) 用に別途利用可能なさまざまな拡張モジュール、およびプラグアンドプレイグローブコネクター周辺機器 (センサー、アクチュエータなど) が付属しており、幅広いユースケースに対応しています。

#### 幅広いソフトウェアフレームワークをサポート
- AWS IoT EduKit のリファレンスハードウェアは、幅広いアプリケーションフレームワーク (FreeRTOS、Arduino、MicroPythonなど) をサポートしているため、デベロッパーは好きな言語でコーディングすることができ、単一のハードウェアプラットフォームでクラウド接続された組み込みアプリケーションの構築を進めることができます。
- AWS IoT EduKit は、EspressifのRainMaker プラットフォームと PlatformIO 開発プラットフォームを通じて、AWS IoTへのオンボーディングを簡素化します。Espressif RainMaker プラットフォームによって、AWS アカウントなしで、リファレンスハードウェアキットの制御、およびスマートホームアプリケーションの制御を行うことができます。PlatformIO 開発プラットフォームは、組み込み開発を簡素化し、コードをすばやく表示、編集、フラッシュすることができます。

#### サンプルコードへの簡単なアクセス
- AWS IoT EduKit は、最も一般的な IoT ソリューションに最新のコンテンツとサンプルコードを提供します。
- AWS IoT EduKit [チュートリアル](/jp/getting-started.html)を通じて、デベロッパーは、さまざまな無料コンテンツにアクセスして、AWS サービスを使用した IoT アプリケーションの構築・管理に関する専門知識を得ることができます。

### 使い方
開始するために、当社の製造パートナーであるM5Stack社から、リファレンスハードウェアキットを、国内で取り扱っている[スイッチサイエンス](https://ssci.to/Core2_for_AWS)のオンラインショップもしくは、直接 [M5Stack ストア](https://m5stack.com/products/m5stack-core2-esp32-iot-development-kit-for-aws-iot-edukit) で購入してください。

ハードウェアキットを受け取ったら、[開始方法](/jp/getting-started.html)チュートリアルにアクセスし、手順に従って Espressif RainMaker Mobile App をスマートフォンにインストールします。Espressif RainMaker Mobile App を利用することで、AWS IoT の EduKit リファレンスハードウェアの制御と、AWS IoT への接続ができるようになります。次に、AWS IoT EduKit のウェブサイトに掲載されている利用可能な無料プロジェクトの一覧から選択します。最初は基本的なコネクテッドホームアプリケーションの構築から始めて、その後は、Amazon SageMaker Autopilot を使った機械学習モデルの実行、または AWS IoT (AFI) 用 Alexa Voice Service Integration を使った Voice Assisted Smart Home アプリケーションの構築へと、徐々に進みます。ハードウェア仕様の詳細については、[M5Stack Docs](https://docs.m5stack.com/#/en/core/core2_for_aws) または [Partner Device Catalog](https://devices.amazonaws.com/detail/a3G0h000007djMLEAY) を参照してください。

---
{{% button href="https://github.com/m5stack/Core2-for-AWS-IoT-EduKit/issues" icon="fas fa-bug" %}}Report bugs{{% /button %}} {{% button href="https://github.com/aws-samples/aws-iot-edukit-tutorials/discussions" icon="far fa-question-circle" %}}Community support{{% /button %}}