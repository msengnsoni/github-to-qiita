---
title: "【対策メモ】AWS認定SysOpsアドミニストレーター-アソシエイト試験対策メモ"
topics: ["AWS", "AWS認定試験", "初心者", "資格"]
published: true
---

# はじめに

本記事は、AWS-SOA試験の対策をしている上で覚えておくべきだと感じたことを
メモとして残しておくものです。
主に問題演習中に連続して間違えたものを記載しています。
SOA試験に関する記事は[こちら](2021-01-17_AWS_da6faebbf934e11bc643.md)

# AWS CLI

- Auto Scalingグループを削除したい場合
最小サイズと希望する容量を0に　→　delete - auto -scaling - group コマンドを使用

# EC2

<img width="80" alt="mojikyo45_640-2.gif" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/813b5402-ce1a-bfcd-1477-f7a901f54666.png">

- インスタンスタイプ
R - RAMのR メモリが必要
C - CPUのC 高性能なCPUが必要
M - MidiumのM バランスの取れたインスタンス郡
I - I/OのI またはInstance storrageのI 主にデータベース用
G - GPUのG ビデオレンダリングもしくは機械学習用
T2/T3 - バースト可能なインスタンス　無制限のバーストが可能(クレジットの制限がない)
- Linux Amazon Machine Imagesは、準仮想化（PV）またはハードウェア仮想マシン（HVM）の2種類の仮想化がある。
- Linux準仮想（PV）AMIは、すべてのAWSリージョンでサポートされているわけではない

# VPC

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/975ca7ea-e53f-7b6c-dbc7-3d97ef19bb43.png)

- VPCフローログを使用して[access error]が出た場合

1. フローログのIAMロールに、フローログレコードをCloudWatchロググループに公開するための十分な権限がない
2. IAMロールに、フローログサービスとの信頼関係がない
3. 信頼関係において、フローログサービスがプリンシパルとして指定されていない
のいずれかが原因。

- フローログを作成したあとに構成の変更をすることはできない。フローログを削除して再度ログを作成する必要がある。
- VPCピアリング - 手動でVPCルートテーブルに追加する必要あり

# EBS

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/813c9093-507a-b14e-ee21-c5a2f47875d0.png)

- スナップショット作成中 - 読み書き可能
- 新しいEBSボリューム（スナップショットから作成された）のデータに初めてアクセスすると、レイテンシが大幅に増加する
→EBSボリュームを初期化する or 事前にウォームアップする

# AutoScaling

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/d3198046-b954-3115-3948-0d69a84bc40e.png)

- 競合がある場合 - エラーメッセージ表示
- CLIからデフォルトの設定でグループ作成 - 詳細モニタリングが自動的に有効化
- ec2の再起動はできない

# CloudWatch

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/b016ee15-fd81-c51c-6f2a-67dbcd6e420b.png)

- Aggregateは集計できない
- カスタムメトリクス - 独自のメトリクスを利用可能
- カスタムメトリクス - コンソールからはアップロード不可
- ELBの1分間隔のメトリクス送信　課金なし
- Auto scallingの1分間隔のメトリクス　課金なし
- Route53ヘルスチェックのメトリクス　課金なし

# CloudFormation

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/2e15d2db-ed59-c5b6-d184-efa0fd1dd3b6.png)

- 作成中に一つエラーがおきたら？ - 全てフォールパック
- EC2およびAutoScalingを使用し、一つのインスタンスが起動するまで次のインスタンスを待機させたい - creationPolicyをリソースに関連付ける
[resource import]を使用して既存のリソースをCloudFromation管理に取り組むことが可能
- AMIにCloudWatchエージェントが含まれている場合、EC2 AutoScalingグループを作成するとEC2インスタンスに自動的にインストールされる
- スタックポリシーは、スタックの更新中にのみ適用され、アクセス制御は提供しません。開発者はIAMポリシーを介してアクセスを提供する必要

# ELB

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/9363c90a-1496-d1ec-e8bc-d0a3b7ac3753.png)

- いつでもAZを追加可能
- アクセスログで取得不可 - フロントエンドの処理時間
- クライアント - elb間はipv4でもipv6でも可能
- elb - ec2間の通信はipv4のみ

# SNS

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/f6d40592-1be4-0567-f1fb-a68afad54729.png)

- SESを送信先には出来ない

# Elastic Beanstalk

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/f041d944-2276-9d59-86d8-0d69b83c71f5.png)

- Perlはサポート外

# SQS

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/e7e3461c-05f2-881d-c010-2161766cf150.png)

- デフォルトキューで順序性を保証したい - 各メッセージにシーケンスを使用する

# Opsworks

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/6b9fb5c2-aaa2-84ed-dd91-220b70207fb6.png)

- トラブルシューティング方法 - インスタンスにログイン

# S3

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/a580e35d-efdc-f469-365c-7ab93583c7c5.png)

- 正常に格納されたかチェックする方法 - HTTPステータスコード200が返ってくるのを確認orMD5チェックサムにて損傷を検出
- マルチパートアップロードが途中終了した場合のパートファイル処理方法 - ライフサイクルポリシーがベストプラクティス
- MFA delete を有効化する方法 - ルートアカウントとAWSCLI
- 各ファイル毎にパスワードを設定する方法はない
- オブジェクトにフェデレートを用いてアクセスさせるユースケース
- ユーザが企業に所属している - SAMLを利用してssoできるようにフェデレート
- ユーザが一般ユーザ - cognitoを利用してフェデレート
- Clacierに保存するためには、S3に一旦保存→Glacierに保存という手順がいる
- 「x-」で始まるクエリ文字列パラメーターを無視する

# DirectConnect

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/b13b0cec-43f1-f591-0722-13638871d8eb.png)

- パブリック仮想インスタンスとプライベート仮想インスタンスのイメージ
→[公式ドキュメント](https://docs.aws.amazon.com/ja_jp/directconnect/latest/UserGuide/Welcome.html)

# AWS System Manager

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/cb26a0ad-1e36-52d3-eb91-d8e9f2f6cb42.png)

- Session Managerを使用すると、インバウンドポートを開いたり、踏み台ホストを維持したりせずにインスタンスに接続可能

# AWS KMS

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/a5d1d6b5-f2fb-8c21-4a3a-276a5daaa701.png)

- カスタマーマスターキー(CMK)の自動ローテーションを有効にすると、暗号化マテリアルを毎年作成し、
- 古い暗号化マテリアルを永続的に保持する。

# X-Ray

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/566f02ba-a4f9-4aee-26ee-93b9bb836bf2.png)

- 統合されていない主要サービス - ELB

# AWS Secrets Manager

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/4e83b41b-354a-d554-a76d-8857f418cf2b.png)

- AWS Secrets Manager は、アプリケーション、サービス、IT リソースへのアクセスに必要なシークレットの保護を支援する
