---
title: "【合格体験記】Certified Kubernetes Application Developer (CKAD)"
topics: ["Kubernetes", "ckad", "資格"]
published: true
---

# 目次

- [目次](#目次)
- [本記事の概要](#本記事の概要)
- [試験の申し込み](#試験の申し込み)
- [使用した教材](#使用した教材)
- [勉強の流れ](#勉強の流れ)
  - [Udemy (+kodekloud)](#udemy-kodekloud)
  - [CKAD Exercieses](#ckad-exercieses)
  - [KillerCoda (Playground)](#killercoda-playground)
- [試験当日の流れと注意点](#試験当日の流れと注意点)
  - [1.専用アプリダウンロード＆起動](#1専用アプリダウンロード起動)
  - [2.身分証明書と顔写真のアップロード](#2身分証明書と顔写真のアップロード)
  - [3.試験環境の撮影](#3試験環境の撮影)
  - [4.試験開始](#4試験開始)
- [試験結果](#試験結果)
- [受験後の感想](#受験後の感想)

# 本記事の概要

Certified Kubernetes Application Developer(CKAD)を受験し取得したので勉強方法を備忘録として投稿します。
CKADを受けようと思っている方や、現在勉強している方がこの記事を参考の一つにしていただけたら幸いです。
なお、本記事はCKA取得済みの状態での体験記になります。ご了承ください。

# 試験の申し込み

私が申し込みしたのはCKAD-JPで、試験前に割とやることがあります。
LPI-JAPANの[公式サイト](https://lpi.or.jp/k8s/exam/)の案内を参照し各登録作業をしましょう。

| 試験名                | 価格     |
| --------------------- | -------- |
| CKAD                   | $395     |
| CKAD-JP                | $430     |
| CKAD-JP(LPI-JAPAN経由) | 55,000円 |

CKA受験時と同様にCKAD-JP(LPI-JAPAN経由)で申し込みました。

# 使用した教材

- Udemy
  - [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)
- [CKAD Exercises](https://github.com/dgkanatsios/CKAD-exercises)
- [KillerCoda](https://killercoda.com/killer-shell-cka) (Playgroundのみ利用)

# 勉強の流れ

勉強期間は2週間です。CKA取得後すぐにCKAD対策を始めたのでこの期間ですが、初学者の方はもっと必要だと思います。

## Udemy (+kodekloud)

CKAで理解をあいまいにしていた箇所や、範囲外だった箇所のみを実施しました。
絞って学習した具体的な箇所は以下になります。

- LivenessProbe, ReadinessProbe
- Job, CronJob
- Labels, Selectors, Annotaions
- Deployment Strategies
- StatefulSet
- Headless Service
- Admission Controllers
- API
- Custom Resoruce Definition
- Custom Controllers
- Helm
- Podman

## CKAD Exercieses

頭から最後まで通すことはせずに、全範囲の問題文を見てパッと答えが浮かばない問題に限りKillerCodaのPlayground環境を利用し実施しました。

## KillerCoda (Playground)

CKA受験時もお世話になりました。手軽に動作確認や問題演習ができるのでありがたいです。

# 試験当日の流れと注意点

全体の流れは以下の通りです。

1. 専用アプリダウンロード＆起動
2. 身分証明書と顔写真のアップロード
3. 試験環境の撮影
4. 試験開始

CKA受験時とは違い、今回は受験前に**大きな**トラブルは起きませんでした。(小さなトラブルはありました笑)

## 1.専用アプリダウンロード＆起動

PSIの専用アプリをダウンロードし起動します。起動時、不要なアプリがある場合はプロセスを落とせと指示されます。
今回はここで少しトラブルが起きました。
MacBookで受験したのですが、PSIのアプリインストール時にエラーとの表示が...
かなり焦りましたが、PSIアプリのインストール画面に「インストール時にトラブルが起きた場合は**ここ**をクリック」という箇所があるのでそこを押したら別バージョン？のインストーラがダウンロードされ、それを使ってインストールしたらうまくいきました。(良かった～～～)

## 2.身分証明書と顔写真のアップロード

アプリ内から身分証明と顔写真をwebカメラで撮影し提出、そのあとから試験管とのチャット(英語)が始まります。
身分証明はパスポートを利用しました。

## 3.試験環境の撮影

試験環境の四方の壁、テーブルの上と下、イス、テーブルにおいているペットボトル、両手のひら、携帯電話、携帯電話を手の届かない場所に置きその様子を見せてとチャットで指示されるので実施します。(一気にではなく順繰りに指示されます)

## 4.試験開始

今回の準備はスムーズに進み、試験開始ボタンを押してから10分程で試験が始まりました。

試験問題について、今回は全16問でした。
難易度のバラつきもCKAの時ほどはっきりしておらず、難しい問題と簡単な問題が斑に配置されていました。

公式サイトを探してもyaml記載例が見当たらないパラメータ、コマンド(podman,helm)が登場します。
焦らずに``kubectl explain``や``podman --help``、``helm help``を使って読み解きましょう。
helmについてはサイトを見ることを許可されていますが、普段からサイトを見慣れていないこともあり、helpコマンドから使い方を参照しました。

それ以外の試験中の所感についてはCKA受験時と変わらなかったので割愛いたします。
気になる方は[以前の記事](https://qiita.com/msengnsoni/items/4ce0e61bc1eae750086f)をご覧ください。

# 試験結果

| 受験日    | 合格ライン | スコア | 結果 | 証明バッヂ                                                                             |
| --------- | ---------- | ------ | ---- | -------------------------------------------------------------------------------------- |
| 2023/9/15 | 66%        | 84     |pass  | [credy](https://www.credly.com/badges/cd6aa92c-1e85-4a80-bd3a-e1a8860c27f7/public_url) |

# 受験後の感想

試験の難易度について、個人的にはCKA < CKADでした。
ただし、これは勉強期間を2週間と設定したりとかなりなめてかかった影響が大きいのかなと思います。
CKAと同じく2か月程期間を設けていれば違った感想になったかもしれません。

CKAを持っていれば新たにインプットする知識量は少ないので、私と同じようにまずCKAを取得された方はさっさとCKADも受けてしまうことをオススメいたします。

ここまで来たらCKS取得も目指したいですね。難易度が跳ね上がるらしいので頑張りたいです。

この記事を開いていただき、
また、ここまで読んでいただきありがとうございました。
