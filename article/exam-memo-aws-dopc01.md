---
title: "【対策メモ】AWS認定DevOpsエンジニア-プロフェッショナル試験対策メモ"
topics: ["AWS", "AWS認定試験", "資格"]
published: false
---

# はじめに

本記事は、AWS-DOP試験の対策をしている上で覚えておくべきだと感じたことをメモとして残しておくものです。
主に問題演習中に連続して間違えたものを記載しています。

- CI/CD
  - Continuous Delivery vs Continuous Deployment
    - Continuous Delivery
      - デプロイを**承認**後に実行する
    - Continuous Deployment
      - デプロイが完全に自動化される。(push = deploy)

- CodeCommit
  - mainブランチにpushさせたくない場合、IAM Policyを用いて制御可能
  - ストリーム対象にはSNSやLambdaを利用可能
  - トリガーを作成するとCodeStarが自動でEventBridgeを作成する

- CodeBuild
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

- CodeDeploy
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

- CodePipeline
  - 検出オプション
    - CloudWatch Event
      - 変更が発生したときに自動的にパイプライン処理を開始する
    - CodePipeline
      - 変更を定期的に確認する
  - 異なるリージョンへのデプロイを設定可能
  - ステージの実行順序パラメータには順次と並列がある

- CloudFormation
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
  - 