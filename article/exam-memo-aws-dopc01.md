---
title: "【対策メモ】AWS認定DevOpsエンジニア-プロフェッショナル試験対策メモ"
topics: ["AWS", "AWS認定試験", "資格"]
published: false
---

# はじめに

本記事は、AWS-DOP試験の対策をしている上で覚えておくべきだと感じたことをメモとして残しておくものです。
主に問題演習中に連続して間違えたものを記載しています。

# CI/CD

- Continuous Delivery vs Continuous Deployment
  - Continuous Delivery
    - デプロイを**承認**後に実行する
  - Continuous Deployment
    - デプロイが完全に自動化される。(push = deploy)

# CodeCommit

- mainブランチにpushさせたくない場合、IAM Policyを用いて制御可能
- ストリーム対象にはSNSやLambdaを利用可能
- トリガーを作成するとCodeStarが自動でEventBridgeを作成する

# CodeBuild

- buildspec.yml
  - phasesセクション
      1. install
      2. pre_build
      3. build
      4. post_build
  - artifactsセクション
    - 成果物をS3バケットにアップロードできる
- 環境変数の値にパラメータストアの値を指定可能
  - ただし実行ロールにSSMのリード許可ポリシーが必要
- 手動承認が必要なユースケースのイベントシーケンス
  1. デプロイ
  2. テスト
  3. (成功した場合)手動で承認の通知

# CodeDeploy

- Deployment settings
  - AllAtOnce
    - 一つずつインスタンスをダウンさせて更新しそれをもとに戻してうまくいったら次のインスタンスに進む
  - HalfAtATime
    - インスタンスの半分をダウンさせてから更新し元に戻す
  - AllAtOnce
    - 一度に全部を更新する
  - Custom
    - 最低限残すインスタンスをPercentage指定もしくは実数値で指定可能
- 閾値によるロールバック
  - CloudWatchアラームを指定可能
  - デフォルトでは閾値に達した際はデプロイが停止される
  - 閾値によるロールバック設定をしていた場合最後の有効なバージョンにロールバックされる
- オンプレミスインスタンスをデプロイ対象にする条件
  - IAM userによるリクエスト認証
    - 作成するオンプレミスインスタンス毎にIAM userが必要
  - IAM role取得によるリクエスト認証
    - AWS STSを利用する
    - 最も安全な方法
- デプロイ進行中にASGスケールアップが発生した場合、最後にデプロイされたアプリケーションで更新される
- appspec.yml
  - hooks
    - デプロイのライフサイクルイベントをフックして差し込む1つ以上のスクリプトを指定
    - ApplicationStop
      - 前回のリビジョンのシャットダウンが開始したとき
    - BeforeInstall
      - CodeDeployがfilesセクションで指定された場所にファイルをコピーする前
    - AfterInstall
      - CodeDeployがfiles セクションで指定された場所にファイルをコピーした後
    - AfterAllowTestTraffic
    - BeforeAllowTraffic
    - AfterAllowTraffic
    - ApplicationStart
      - アプリケーションのリビジョンが開始する直前
    - ValidateService
      - サービスの検証が終わった後

# CodePipeline

- 検出オプション
  - CloudWatch Event
    - 変更が発生したときに自動的にパイプライン処理を開始する
  - CodePipeline
    - 変更を定期的に確認する
- 異なるリージョンへのデプロイを設定可能
- ステージの実行順序パラメータには順次と並列がある
- ステージング環境をデプロイする前にCodeBuildテストをステージとして持つことはできない

# CloudFormation

- SSM Parameter Storeから値を取得可能
- DependsOn
  - DependsOnで指定したリソースの後に作成される(指定したリソースがない限り作成されない)
- S3からファイルを取得しリソースを作成した際、S3ファイルを更新後にCFN側に知らせる方法
  - S3のバージョニングを有効にし、バージョンIDをParametersで指定する
- ドリフト機能
  - 作成したテンプレートの内容が変更された場合検出可能(ドリフト機能で修正は不可)
- cfn-hup
  - リソースメタデータが変更されたことを検知し、指定した操作を実行するデーモン
- スタックポリシー
  - スタック内で個別にリソースを保護可能
  - 更新中のスタックポリシーの変更は更新時の一時的なものになるため、スタック本体のスタックポリシーには影響しない
- ReplacingUpdate
  - 更新時インスタンスの総数が維持される
  - Auto ScalingグループまたはAuto Scalingグループ内のインスタンスが更新時に置換される
  - 展開失敗時は古いグループにロールバックし作成した新しいグループを削除する
- RollingUpdate
  - 一台ずつ順番に更新する

# ElasticBeanstalk

- configrationファイル以外に設定を変更する
  - eb config saveコマンドによる保存された設定
  - .ebextantionsフォルダによる設定
  - 優先順位は以下の通り
       1. 直接手動で設定(GUI)
       2. 保存された設定
       3. ebextensionフォルダ
       4. デフォルト設定
- 結局はCFNで構築されることを忘れない
- .ebextensionsフォルダで定義したリソースはEBと紐づいているためEBが削除されると同様に削除される
- commandsとcontainer_commands
  - commandsは何よりも先にアプリケーションやwebサーバ内で実行されるコマンドを定義する
  - container_commandsはアプリケーションやwebサーバがセットアップされた後に実行されるコマンドを定義する

# API Gateway

- エンドポイントタイプ
  - リージョン
    - 今いるリージョンに配置
  - エッジ最適化
    - クラウドフロントエッジに配置
    - レイテンシーを削減したい場合に選択
  - プライベート
    - VPC内に配置されVPC内のリソースにアクセス可能
    - API GatewayのVPCエンドポイントを介してのみアクセス可能
- Canary
  - APIGateway自体の機能
  - 特定の重さを設定し変更を確認できる
  - ステージング環境として使用
  - 特定の機能と統合されている場合(Lambda等)はLambda自体の機能(Aliases機能)でcanaryを実現可能

# CloudTrail

- ログを特定のアカウントに集約可能(マルチアカウントロギング)

# CloudWatch Logs

- ストリーム可能なサービス
  - Lambda
  - Kinesis
  - Kinesis Firehose
  - OpenSearch Service (new!)

# CloudWatch Event

- S3 Event vs CloudWatch Event
  - S3 Eventは基本的なイベントに対してアクションを設定可能
  - S3サービスにネイティブである(特段別サービスで設定不要)
  - CloudWatch Eventはより多くのイベントで設定可能
  - (CloudTrailログを操作する場合)CloudTrailサービス側で(CloudWatch Eventにより)ログの操作を可能にする設定が必要

# ElasticSearch (Amazon ES)

- ユースケース
  - ログアナリティクス
  - リアルタイムアプリケーションモニタリング
  - セキュリティアナリティクス
  - フルテキストサーチ
  - クリックストリームアナリティクス
  - 索引

# AWSリソースのタグ付け (Tagging)

- 一般的なベストプラクティス
  - コスト計算のための利用
  - セキュリティ、操作対象リソースの明確化(TBAC)

# Trusted Advisor

- チェックは5分に一回リフレッシュ可能
- 24時間以上チェックが実行されなかった場合自動でリフレッシュされる
- 使用率の低いEC2インスタンスを検知可能
- 

# Auto Scaling Group (ASG)

- ASGグループの作成(起動テンプレートの使用)でバージョン管理可能
- プロセスを一時的に中断して再開する
  - unhealthyのインスタンスのトラブルシューティングのため一時的に削除処理を中断する...等
- 選択可能な中断したプロセスのタイプ
  - Launch
  - Terminate
  - AddToLoadBalancer
  - AlarmNotification
  - AZRebalance
  - HealthCheck
  - InstanceRefresh
  - ReplaceUnhealthy
  - ScheduledActions
- ライフサイクルフック
  - インスタンススケールアウトorスケールイン時に特定の処理をフックとして追加する
  - ユースケース
    - インスタンス削除時にログを退避する...等
- 終了ポリシー
  - スケールイン時の削除対象インスタンスを選ぶポリシーを選択可能
  - デフォルトの終了ポリシー
    1. スケールインから保護されていないインスタンスが存在するAZを決定する
    2. 対象インスタンスの中に最も古い起動テンプレートを使用しているインスタンスを判断する
    3. (起動設定を使用しているインスタンスがある場合は優先して終了する)
    4. 上記適用後にインスタンスが残っている場合、次の課金時間に最も近いインスタンスを終了する
    5. 4.が同じ課金時間の場合はランダムに終了
- スケジュールに基づいたスケーリングを設定可能

# Kibana

- アクセスコントロールオプション
  - Amazon Cognito認証
  - プロキシサーバのある、またはないIPベースのアクセスポリシー

# Amazon Inspector

- 自動化されたセキュリティ評価サービス
- AWSにデプロイしたアプリケーションのセキュリティとコンプライアンスを向上させる

