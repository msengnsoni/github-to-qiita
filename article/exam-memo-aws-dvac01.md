---
title: "【対策メモ】AWS認定デベロッパー-アソシエイト試験対策メモ"
topics: ["AWS", "AWS認定試験", "初心者", "資格"]
published: true
---

# はじめに

本記事は、AWS-DVA試験の対策をしている上で覚えておくべきだと感じたことを
メモとして残しておくものです。
主に問題演習中に連続して間違えたものを記載しています。

# AWS CLI

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/3a8f8ebd-c657-5da7-990b-5182428f0b27.png)

・--dry-run  実行可能でも実行しない。権限設定を確認する際等に使用する。
・(EC2内で) <http://169.254.169.254/> にアクセスするとmeta-dataを取得可能。
・MFA認証をCLIで実施可能
・認証情報を参照する順番

1. CLIオプション --region, --output, --profile
2. 環境変数 AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN
3. credentialsファイル ~/.aws/credentials (linux mac) C:/Users/user/.aws/credentials (win)
4. configファイル　/.aws/cofig (linux mac) C:/Users/user/.aws/config (win)
5. コンテナのcredentials
6. EC2のInstance profile credential

# AWS SDK

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/a82d0c13-6af1-b885-d991-2f055ed208ee.png)

・認証情報を参照する順番

1. 環境変数
2. java system properties
3. デフォルトのcredential profileファイル
4. コンテナのcredentials
5. EC2のInstance profile credentials

# ECS

<img width="80" alt="mojikyo45_640-2.gif" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/813b5402-ce1a-bfcd-1477-f7a901f54666.png">
・ECS Task PlacementはECS with EC2のみ使用可能
3つのタイプ
Binpack - 使用するインスタンスの数を最小にしようとする most cost effective
Ramdom - タスクをランダムに配置する
Spread - インスタンスIDやAZに基づいてタスクを分散配置する
2つの制約
distinctInstance - それぞれのタスクは異なるコンテナインスタンスに配置する
memberOf - クラスタークエリ言語を使用して制約を定義可能     example:t2のインスタンスにのみタスクを配置する

# Elastic Beanstalk

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/f041d944-2276-9d59-86d8-0d69b83c71f5.png)
・環境でELB作成したあとはELBのタイプを変えることはできない。
・EB with Docker
シングルDockerの場合、コンテナを実行可能だが、ECSは使用しない。
マルチDockerの場合、ECSを使用する。
•環境のプラットフォームバージョンはコンソールまたはEBCLIから変更できる

# CodeCommit

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/9513e85d-ed9b-80d3-87c9-961ddc13b6b2.png)

・AWS SNS、Lambda、CloudWatch Event Rulesを使用して通知が可能。
・クロスアカウントアクセス有

# CodeBuild

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/a6dafafb-963f-b9ce-a91e-066b7437fb25.png)

・フックの順序
アプリケーションストップ
↓
バンドルをダウンロード
↓
beforeInstall
↓
afterInstall
↓
アプリケーションスタート
↓
サービスの検証
・CodeBuildコンテナは実行終了時に削除される
・Dockerイメージは、プライベートレジストリからのものも使用できる

# CodeDeploy

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/0f246563-8adb-83e7-333c-1b4bbcc8402e.png)

Appspec.yamlファイルの必須プロパティ
name, alias, currentversion, targetversion

# X-Ray

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/566f02ba-a4f9-4aee-26ee-93b9bb836bf2.png)
・BeanStalkでX-Ray デーモンを有効にする方法 - .ebextensions/xray-daemon.configを作成する
•サービスマップは個々の地域のデータではなく全ての地域のデータが表示される
•SQSと統合されている場合、リソースヘッダーはSQSのメッセージサイズ等に影響しない
•サンプリングルールのデフォルト - 1秒あたりに1つのリクエスト、ホスト毎の追加の5%の要求がリクエストされる
・X-Ray SDKが提供するもの -
インターセプター コードに追加して受信 HTTP リクエストをトレースする
クライアントハンドラー アプリケーションが他の AWS サービスの呼び出しに使用する AWS SDK クライアントを計測する
An HTTP クライアント 別の内部および外部 HTTP ウェブサービス呼び出しを計測する

# Lambda

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/a6a5af2a-487e-7368-4897-11f2720bb41d.png)

・タイムアウトは最大15分
・環境変数は最大4KB
・FunctionName - 関数の名前
・Layer - Lambda関数実行環境に追加された関数レイヤーのリスト
・Environment - Lambda関数実行中にアクセス可能な変数
・Handler - 関数を実行するために呼び出すコード内のメソッドの名前
・関数をスケジューリングするベストプラクティスはCloudWatch Iventを使用すること

# DynamoDB

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/5acebb44-6e23-a53a-252f-c5d63f72df7c.png)

・WCU(Write Capacity Units)計算方法例
*2KBのオブジェクトを毎秒10個書き込む - 2*10 = 20WCU
*4.5KBのオブジェクトを毎秒6個書き込む - 5*6 = 30WCU
*2KBのオブジェクトを毎分120個書き込む - 2*(120 / 60) = 4WCU

・RCU(Read Capacity Units)計算方法例
*4KBの10個のアイテムを強力な整合性のある毎秒読み込みで読み込む - (4 / 4)* 10  = 10RCU
*12KBの16個のアイテムを結果整合性のある毎秒読み込みで読み込む - ( 12 / 4 )* ( 16 / 2 ) = 24RCU
*6KBの10個のアイテムを強力な整合性のある毎秒読み込みで読み込む - ( 8 / 4 )* 10= 20RCU

・グローバルセカンダリインデックスでは一貫性のある読み取りをサポートしていない
・一貫性のある読み取りをDAXクラスターにリクエストした場合、結果はキャッシュされない
•DynamoDBストリームとLambdaトリガーの併用はベストプラクティス

# CloudWatch

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/b016ee15-fd81-c51c-6f2a-67dbcd6e420b.png)
・基本的なモニタリングにアラームを設定する場合 - 少なくとも5分
・詳細なモニタリングにアラームを設定する場合 - 少なくとも1分

# S3

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/a580e35d-efdc-f469-365c-7ab93583c7c5.png)
・バケット内のプレフィックスごとに1秒あたり3500のPUTリクエストと、5500のGETリクエストを提供する

# IAM

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/fee45474-b68d-beaf-72a4-62980f0e0216.png)

・バージョニング機能あり
・明示的に拒否するポリシーがある場合、他の許可されたステートメントを上書きする

# KMS

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/a5d1d6b5-f2fb-8c21-4a3a-276a5daaa701.png)
・デフォルト設定では、対象CMKをサポート
