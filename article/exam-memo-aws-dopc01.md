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
    - 
