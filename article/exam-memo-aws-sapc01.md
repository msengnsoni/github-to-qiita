---
title: "【対策メモ】AWS認定ソリューションアーキテクト-プロフェッショナル試験対策メモ"
topics: ["AWS", "AWS認定試験", "資格"]
published: true
---

# はじめに

本記事は、AWS-SAP試験の対策をしている上で覚えておくべきだと感じたことを
メモとして残しておくものです。
主に問題演習中に連続して間違えたものを記載しています。

# IAM

- IAMロールによる権限委任に必要な3要素
    1. 被委任側のAWSアカウントID
    2. ロールに紐づける外部ID(External ID)
    3. 被委任側がAWSリソースを操作するためのアクセス権

# AWS Directory Service

- Managed Microsoft AD
  - AWS上に独自のADサービスを構築するマネージドサービス
  - オンプレ側ADと信頼関係を結ぶことができる
- Simple AD
  - AWS側でADサービスを使用できるマネージドサービス
  - オンプレ側ADと繋がることはできない
- AD Connector
  - オンプレ側のADと連携する際に使用するサービス

# AWS Organizations

- 一括請求と同様の請求情報の統合を自動的に使用できるようになる

# CloudTrail

- 特定のログをフィルターしアラートを上げる機能は、CloudTrail単体では実現不可
  - CloudTrail→CloudWatchLogsにstream→Metric Filtersでフィルタ→CloudWatchAlarmをコール→SNSで通知等が必要

# KMS

- 複数リージョンでキーを同期(複製)した場合、キーマテリアルは同一のものになる

# RDS - Security

- Transparent Data Encryption(TDE)は、Oracle、SQL Server用
- IAM認証は、MySQL、PostgreSQL用
- CloudTrailでは追跡不可

# VPC

- リージョン間VPCピアリングでリージョン間の通信が可能
- インターリージョンVPCペアリング
    > インターリージョンVPCピアリングとは、異なるAWSのリージョンにあるVPC同士を、特別なゲートウェイやVPN接続などをすることなく、
    >
    > AWSのバックボーンネットワークを通じてプライベートIPアドレスで互いにやり取りできる機能です。
    >
- ゲートウェイ型エンドポイント
  - S3 と DynamoDBが使用可能
- インターフェイス型エンドポイント
  - その他のAWSサービス
- CIDRの拡張方法 - セカンダリCIDRブロックを追加

# EC2

- ネットワーク帯域は、インスタンスタイプによって決まる
- プレイスメントグループに追加または削除するにはインスタンスが停止しCLIで実行する必要がある
- Nitroシステム
       - 次世代のインスタンス基盤
       - コスト削減、セキュリティ強化、新インスタンスタイプの提供等さまざまな利点がある
       - 特定のインスタンスタイプ（C5,M5,R5等）の場合、VPC Traffic Mirroring機能を利用可能

# EC2 - HPC

- EC2拡張ネットワーキングの利用方法
  - ENA(Elastic Network Adapter)を使用、100Gbpsの帯域を実現可能
  - インテル82599VFを使用、最大10Gbpsの帯域を実現可能だがレガシー
- **EFA(Elastic Fabric Adapter)**
  - HPC専用の改良されたENA
  - Linuxでのみ機能する
  - ノード間通信、密結合のワークロードに最適

# Lambda

- 実行時間は--最大15分--のため、実行時間が長いアプリケーションの代替手段にはならない。
- より良いCPUが必要な場合はRAMを増やす
- Lambda DLQを呼び出し可能なサービスは非同期であるSNS

# API Gateway

- タイムアウトは29秒
- ECS/Fargate構成よりも、API Gateway + Lambda構成のほうがコスト面で優れている
- APIキャッシュ
  - 指定されたステージでキャッシュを有効にできる
  - API Gateway でキャッシュを有効にするには、専用のキャッシュインスタンスを作成する。最長4分かかる。

# S3

- リクエスタ支払い
  - 設定することでバケットへのリクエストおよびデータのダウンロードにかかるコストをリクエスタが支払うように変更可能。ただし、データ容量に対するコストは常にバケット所有者が支払う

- ダウンロードのパフォーマンス改善 = Cloud Front
- アップロードのパフォーマンス改善 = S3転送アクセラレーション & S3マルチパートアップロード
- 最大ファイルサイズは5TB
- S3転送アクセラレーションは、DXリンクを使用したアップロードよりも速く転送できる

# EBS

- パフォーマンス(IOPS)を向上したい場合は容量を追加する

# CloudFront

- オリジンフェイルオーバー機能により、500,504エラーが返された際セカンダリオリジンにリクエストをルーティンできる
- Lambda@Edge機能により、可用性を向上できる
- 基本的な性能はCloudFront Functions < Lambda@Edge
- CloudFront Functions = JavaScript
- Lambda@Edge = Node.js, Python

# ElastiCache

- Redis vs Memcached
  - Redis
    - オートフェイルオーバー付きマルチAZ
    - リードレプリカを持ち高可用
    - バックアップとリストアが可能
  - Memcashed
    - シャーディングのためのマルチノード
    - 可用性はない
    - バックアップとリストアが出来ない
    - マルチスレッド構成

# DBサービス種別

- RDBMS
  - RDS、Oracle Database
- NoSQL
  - DynamoDB、ElastiCashe
- DWH
  - --Redshift--

# DynamoDB

- Auto Scalingをサポートしている
- RCUのデフォルト上限は40000
  - それ以上の設定値にする場合はAWSサポートに上限緩和申請をする

# RDS

- 読み込み負荷改善 = リードレプリカやキャッシュの追加
- 書き込み負荷改善 = インスタンスタイプを上げる
- 暗号化されたDBの暗号化を解除する方法は再作成しかない

# Kinesis

- Kinesis Data Streams
  - Kinesisを構成するサービスの一つ
  - ストリームデータを、  Kinesis Client Libraryなどで独自に実装したリアルタイムアプリケーションや、Kinesis Data Analytics等に--配信--する
- Kinesis Data Firehose
  - Kinesisを構成するサービスの一つ
  - 様々なProducer(Kinesis Data Streams、CloudWatch Logs、CloudWatch Events等)からロードしたストリームデータを、S3、Redshift、Elasticsearch Serviceといったデータレイク、データストア、分析ツールや3rdパーティへ送信する
- Kinesis Data Analytics
  - Kinesisを構成するサービスの一つ
  - 標準的なSQLクエリを使用して、ストリームデータを--リアルタイムに分析--する
- Kinesis Video Streams
  - 接続されたデバイスの動画を、Recognition Videoなどのリアルタイム動画分析機能へ配信する
- Kinesis Client Library
  - Kinesis Data Streamsに流れるレコードを受信して処理を行うコンシューマアプリケーションを実装するためのAWS製ライブラリ

# Redshift

- クロスリージョンスナップショット機能
  別のリージョンにスナップショットを転送する、Redshiftの可用性を担保
- 他サービスでは使用可能だがRedshiftには存在しない機能
    1. グローバルテーブル (DynamoDB)
    2. リードレプリカ (RDS)
- 集計に特化。保存速度は速くないため、リアルタイム性が重視されるデータ集計には向かない

# Athene

- S3をデータソースとして、QuickSightと併用可能
- パフォーマンスを最適化するための推奨事項
  - Apache Parquet や Apache ORC などの列形式でのデータ格納
  - 日付を含むキーを使用した Amazon S3 での Apache Hive パーティショニングの使用

# Aurora

- グローバルデータベースを持つ

# Elastic Beanstalk

- コスト最適
  - ECSと比較した場合、リソースの有効利用という観点で若干劣る(ワークロードにもよる)

# OpsWork

- Stack, Layer, Recipeの関係
  - Stack
    - システムの環境定義単位
    - 1つ以上のLayerから構成される
  - Layer
    - ロードバランサー層、アプリケーション層、データベース層で分離する
    - 各Layerのライフサイクルイベントが発生したときに実行するRecipeを割り当てる
  - Recipe
    - chefレシピ
- ライフサイクルイベントの種類
  - Setup
  - Configure
  - Deploy
  - Undeploy
  - Shutdown

# CloudFormation

- DeletionPolicy 属性によりスタックが削除されてもリソースを保持できる

# Systems Manager - SSM

- AWS-DefaultPatchBaseline
  - --Windows機--に対して適用するデフォルトのパッチベースライン
- AWS-RunPatchBaselineコマンド
  - パッチベースラインを実行するコマンド、--LinuxとWindows--両方に使用可能

# Strage Gateway

- ボリュームゲートウェイの種類
    1. Gateway-Stored Volumes
        オンプレミスのデータのスナップショットを非同期にS3にバックアップする。
        データセット全体への--低レイテンシーアクセスが必要な場合--には、保管型のボリュームゲートウェイを使用する。
    2. Gateway-Cached Volumes
        データセット全体をS3に保存し、頻繁にアクセスするデータサブセットのコピーをオンプレミス側に保存する。
        オンプレミスのストレージの--拡張コストを抑えることができる。--

# Direct Connect

- Direct Connect Gateway
  - Direct Connectおよびプライベート仮想インターフェイス（VGW）を使用して、複数のVPCや複数のリージョン間を接続することができるサービス
- パブリック仮想インターフェイス
  - S3、Glacier等の非VPCサービスに接続する
- プライベート仮想インターフェイス
  - VPCに接続する

# PrivateLink(VPC Endpoint Services)

- パブリックなネットワークを通じずに、VPC間の通信を可能にする
  - ユースケース：AWS上に作成されたSaaSとVPC間を、ゲートウェイサービスを使わずセキュアに通信したい

# AWS CloudSearch

- PDFファイル等のインデックス化とテキスト検索機能を提供するマネージド型サービス

# Step Function

- 複数のサーバーレス関数を調整するために使用される

# Snowball

- データソースからSnowballへの転送速度解決方法（パフォーマンスへの影響度が高い順）
  1. 最新の Mac または Linux Snowball クライアントを使用する
  2. 小さなファイルを一緒にバッチ処理する
  3. 一度に複数のコピー操作を実行する
  4. 複数のワークステーションからコピーする
  5. ファイルではなくディレクトリを転送する

# AWS SSO

- モバイルアプリケーションには利用できない

# CodeStar

- 多様な開発言語やフレームワークに応じたCI/CDパイプラインを構築可能にする

# AppStream 2.0

- アプリケーション及びデスクトップストリーミングサービス。UXの問題を解決できる

# AppSync

- 複数のデータベース、マイクロサービス、および API を照会するために使用できる GraphQL API を提供する
- 1つ以上のデータソールからデータにアクセスでき、操作統合のためのAPIを作成できる

# Glue

- 分析、機械学習、アプリケーション開発のためのデータの検出、準備、結合を簡単に行える、サーバーレスデータ統合サービス
