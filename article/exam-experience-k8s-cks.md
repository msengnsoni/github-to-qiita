---
title: "【合格体験記】Certified Kubernetes Security Specialist (CKS)"
topics: ["Kubernetes", "cks", "資格"]
published: true
---

# 目次

- [目次](#目次)
- [本記事の概要](#本記事の概要)
- [試験の申し込み](#試験の申し込み)
- [使用した教材](#使用した教材)
- [勉強の流れ](#勉強の流れ)
  - [kodakloud](#kodakloud)
  - [KillerShell](#killershell)
  - [Udemy](#udemy)
  - [KillerCoda](#killercoda)
- [試験当日の流れと注意点](#試験当日の流れと注意点)
  - [1.専用アプリダウンロード＆起動](#1専用アプリダウンロード起動)
  - [2.身分証明書と顔写真のアップロード](#2身分証明書と顔写真のアップロード)
  - [3.試験環境の撮影](#3試験環境の撮影)
  - [4.試験開始](#4試験開始)
- [試験結果](#試験結果)
- [受験後の感想](#受験後の感想)

# 本記事の概要

Certified Kubernetes Security Specialist (CKS)を受験し取得したので勉強方法を備忘録として投稿します。
CKSを受けようと思っている方や、現在勉強している方がこの記事を参考の一つにしていただけたら幸いです。
なお、本記事はCKAとCKADを取得済みの状態での体験記になります。ご了承ください。

# 試験の申し込み

これまでと同様にCKS-JP(LPI-JAPAN経由)で申し込みました。

| 試験名                | 価格     |
| --------------------- | -------- |
| CKS                   | $395     |
| CKS-JP                | $430     |
| CKS-JP(LPI-JAPAN経由) | 55,000円 |

:::note warn
LPI-JAPAN経由の購入について、2023/10/16に価格改定があり62,000円になってます。円安辛いですね...
:::

# 使用した教材

- kodakloud
  - [Certified Kubernetes Security Specialist (CKS)](https://kodekloud.com/courses/certified-kubernetes-security-specialist-cks/)
- [KillerShell](https://killer.sh/cks)
- Udemy
  - [Kubernetes CKS 2023 Complete Course - Theory - Practice](https://www.udemy.com/course/certified-kubernetes-security-specialist/)
- [KillerCoda](https://killercoda.com/killer-shell-cks)

# 勉強の流れ

今回の勉強期間は約1ヶ月半でした。1度目の受験に1ヶ月、2度目の受験で半月期間を設けたというような感じです。

## kodakloud

CKAとCKADでお世話になったMumshad Mannambethさんの教材です。CKSだけ何故かUdemyにないので本家のKodakloudで受講しました。
kodakloudだと字幕が動画内に表示される仕様で翻訳ができず、英語での学習になりました。
そのため教材の内容を完全に理解したかと言われると怪しく、なんとなくこういうことがポイントなんだろうな〜という浅い理解での学習になってしまいました。

## KillerShell

KillerShellで出題される内容はやはり本番よりも難しいと感じました。
1回目はほとんど解けず、最終的に5時間くらいかけて最後まで解いたと記憶しています(笑)
CKAでもCKADでも同様の対策ですが、問題文を見たときにkubernetes.ioのどのドキュメントを見れば回答できるのかを覚えると良いです。

## Udemy

今回は一度試験に落ちてしまったので、補強用の教材として有名なUdemyの講座を受講しました。
最後まで通したわけではなく、あくまで補強という形で特定の分野に絞って学習しました。
具体的絞って学習した分野は以下になります。

- Microservice Vulnerabilities - mTLS
- Supply Chain Security - Image Footprint
- Supply Chain Security - Static Analysis
- Supply Chain Security - Image Vulnerability Scanning
- Supply Chain Security - Secure Supply Chain
- Runtime Security - Behavioral Analytics at host and container level
- Runtime Security - Immutability of containers at runtime
- System Hardening - Reduce Attack Surface

## KillerCoda

これまではplaygroundのみの利用でしたが、今回はいくつかのシナリオを実施しました。
実施した具体的なシナリオは以下になります。

- Apiserver Crash
- Apiserver Misconfigured
- Container Herdening
- Container Image Footprint User
- Falco Change Rule

# 試験当日の流れと注意点

全体の流れは以下の通りです。

1. 専用アプリダウンロード＆起動
2. 身分証明書と顔写真のアップロード
3. 試験環境の撮影
4. 試験開始

前述していますが、今回は1度不合格となり2回目の受験で合格しました。
1度目の受験時は何事もなく終わったのですが、2度目の試験中にネットワークトラブルで試験が中断されてしまいました。
詳細は「4.試験開始」に書きます。

## 1.専用アプリダウンロード＆起動

PSIの専用アプリをダウンロードし起動します。起動時、不要なアプリがある場合はプロセスを落とせと指示されます。

## 2.身分証明書と顔写真のアップロード

アプリ内から身分証明と顔写真をwebカメラで撮影し提出、そのあとから試験管とのチャット(英語)が始まります。
身分証明はパスポートを利用しました。

## 3.試験環境の撮影

試験環境の四方の壁、テーブルの上と下、イス、テーブルにおいているペットボトル、両手のひら、携帯電話、携帯電話を手の届かない場所に置きその様子を見せてとチャットで指示されるので実施します。(一気にではなく順繰りに指示されます)

## 4.試験開始

今回も準備は特段トラブルなくスムーズに進み、試験開始ボタンを押してから10分程で試験が始まりました。

試験問題について、1度目の試験、2度目の試験ともに全16問でした。

ネットワークのトラブルについて、終了20分前くらいに突然帯域が許容以下になった旨のポップアップが表示されPSIのアプリケーションが落ちました。
とても焦りましたが、再度アプリケーションを立ち上げて試験環境に接続したところ無事再開できました。

ただ、試験環境の撮影をもう一度実施しないといけないかつ、その間試験時間はストップしない仕様のようで、戻ってきたときには残り3分くらいになっていました。
2度目の受験だったのでほとんど解き終えていましたが、1度目にこの事象が起きていたらよりスコアが悲惨なものになっていたと思います。

もしこのようなことが起きてしまった場合は、落ちついて再度アプリケーションを立ち上げて接続し、試験管の指示通りにしてなるべく早く試験に復帰できるようにしましょう。

# 試験結果

| 受験日     | 合格ライン | スコア | 結果 | 証明バッヂ                                                                             |
| ---------- | ---------- | ------ | ---- | -------------------------------------------------------------------------------------- |
| 2023/11/7  | 67%        | 66     | fail | -                                                                                      |
| 2023/11/18 | 67%        | 76     | pass | [credy](https://www.credly.com/badges/c31482d5-a718-4bf8-a3f4-4101d207d4c7/public_url) |

1度目の受験は1スコア差で不合格...悔しい！
まったく解くことができない問題が3問ほどあったので、その分野を重点的に学習して2度目に臨みました。

# 受験後の感想

試験難易度は文句なしにCKSが高いと感じました。
様々なツールが登場しますので、各ツールが何を実現したいかを意識しながら学習するとより知識の定着につながると思います。
また、static podのmanifestを弄る問題が多く出題されるので必ずバックアップを取ってから編集するようにしてください。

各ツールのコマンドの使い方がわからなくなっても、``-h``で説明を見れるので慌てずに落ちついて見ましょう。
``kubectl``関連(特にオプションの書き方)が分からなくなったら、確認したいkubectlコマンドの後に``-h | grep kubectl``で主要なオプションの書き方の例を参照できるのでおすすめです。個人的に必須テクニックだと思ってます。

これで一旦はkubernetesの学習は終了とし、今後はGoogle Cloudの学習に手を出したい考えてます。

この記事を開いていただき、
また、ここまで読んでいただきありがとうございました。
