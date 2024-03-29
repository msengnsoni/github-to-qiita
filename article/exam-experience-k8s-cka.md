---
title: "【合格体験記】Certified Kubernetes Administrator (CKA)"
topics: ["Kubernetes", "CKA", "資格"]
published: true
---

# 目次

- [目次](#目次)
- [本記事の概要](#本記事の概要)
- [試験の申し込み](#試験の申し込み)
- [使用した教材](#使用した教材)
- [勉強の流れ](#勉強の流れ)
  - [Udemy (+kodekloud)](#udemy-kodekloud)
  - [Killer Shell](#killer-shell)
  - [Youtube](#youtube)
  - [KillerCoda](#killercoda)
- [試験当日の流れと注意点](#試験当日の流れと注意点)
  - [1.専用アプリダウンロード＆起動](#1専用アプリダウンロード起動)
  - [2.身分証明書と顔写真のアップロード](#2身分証明書と顔写真のアップロード)
  - [3.試験環境の撮影](#3試験環境の撮影)
  - [4.試験開始](#4試験開始)
- [試験結果](#試験結果)
- [受験後の感想](#受験後の感想)

# 本記事の概要

Certified Kubernetes Administrator(CKA)を受験し取得したので勉強方法を備忘録として投稿します。
CKAを受けようと思っている方や、現在勉強している方がこの記事を参考の一つにしていただけたら幸いです。

# 試験の申し込み

私が申し込みしたのはCKA-JPで、試験前に割とやることがあります。
LPI-JAPANの[公式サイト](https://lpi.or.jp/k8s/exam/)の案内を参照し各登録作業をしましょう。

ちなみにですがCKAとCKA-JPには価格差があります。

| 試験名                | 価格     |
| --------------------- | -------- |
| CKA                   | $395     |
| CKA-JP                | $430     |
| CKA-JP(LPI-JAPAN経由) | 55,000円 |

私が購入した日(2023/8/16)の為替レートだと$395は57,000円程だったのでCKA-JP(LPI-JAPAN経由)にしました。

# 使用した教材

- Udemy
  - [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)
- Killer Shell (CKA受験バウチャー購入後に利用可能)
- Youtube
  - [SetTech Systematic - Clarity Brings Serenity!](https://www.youtube.com/playlist?list=PLTYt3kXV0MXj_JrhssXH6voHT2HjtWbBA)
  - [Alok Kumar](https://www.youtube.com/playlist?list=PL6nVblW4NNAQrgSjhT8iK_v7ROIV2ikVu)
- [KillerCoda](https://killercoda.com/killer-shell-cka) (Playgroundのみ利用)

# 勉強の流れ

勉強期間は2か月弱です。ゆるく勉強してこの期間なので、がっつり本腰を入れてやれる方はもっと短く設定しても大丈夫かと思います。(全くの初学者の場合はもっといるかも)

## Udemy (+kodekloud)

他に合格体験記を書いてくださっている皆様と同様、Udemyの教材を頭から最後まで実施します。
私は業務でKubernetesを触っていましたが体系的に学んでこなかったため、
知らない機能を知れたり理解を曖昧にしていた箇所が肉付けされ、とても勉強になりました。
私は1週しかしていませんが、より知識の定着を目指す方はkodekloudを2~3週実施するとよさそうです。

## Killer Shell

その後はCKAを申し込むと2回分実施可能になるKiller Shellを実施します。
ほぼすべての問題が本番より難しいので1回目は時間を気にせず解き、2回目は制限時間内にどれほど解けるかを意識します。
ちなみに私は25問中21問ほどしか解けませんでした泣
なお、Killer Shellはactivate後に36時間問題及び回答にアクセス可能×2回分という形なので、試験直前の土日に実施することをオススメいたします。

## Youtube

私の場合は平日を試験日にしたので、試験日までyoutubeで問題演習の動画を見て記憶を定着させます。
上に挙げた2チャンネルはレベル的には簡単なので、問題文を見たときに頭の中で回答までのプロセスをイメージできれば十分かなと思います。

## KillerCoda

KillerCodaは動作確認を行いたい時に便利です。その他のシナリオも普通に勉強になりそうですが、私は時間がないためパスしました。
余裕のある方は実施するとより安心だと思います。

# 試験当日の流れと注意点

私の場合自宅では[要求される試験環境](https://docs.linuxfoundation.org/tc-docs/certification/lf-handbook2/candidate-requirements)を用意できなかったので、[レンタル会議室](https://www.spacee.jp/listings/6412)を利用しました。
予約した受験日時の30分前から試験環境にアクセス可能になるため、試験前30分+試験時間2時間+バッファ1時間の3時間半程借りておくと安心です。
バッファそんなにいるの？と思う方もいらっしゃるでしょうが、理由は後述します。

この試験はここからが大変です。試験環境にアクセスした後の流れをご説明しますと、

1. 専用アプリダウンロード＆起動
2. 身分証明書と顔写真のアップロード
3. 試験環境の撮影
4. 試験開始

となるのですが、``3.試験環境の撮影``でかなり手間取りました。以下で順番に説明します。

## 1.専用アプリダウンロード＆起動

PSIの専用アプリをダウンロードし起動します。起動時、不要なアプリがある場合はプロセスを落とせと指示されます。

## 2.身分証明書と顔写真のアップロード

アプリ内から身分証明と顔写真をwebカメラで撮影し提出、そのあとから試験管とのチャット(英語)が始まります。
身分証明はパスポートを利用しました。

## 3.試験環境の撮影

試験環境の四方の壁、テーブルの上と下、イス、テーブルにおいているペットボトル、両手のひら、携帯電話、携帯電話を手の届かない場所に置きその様子を見せてとチャットで指示されるので実施します。(一気にではなく順繰りに指示されます)
私はノートPCに備え付けのカメラを使用したので手間取りました。

その後、「カメラを拭いて」とか「明るいところに移動して」とか指示されて確認長いなーとか考えていたら、**映像が暗くてこのカメラじゃ試験できないから画面に表示されてる電話番号(テクニカルサポート)に電話して**と言われてしまいました。(試験環境写している時に言ってよ...！)

その電話番号はもちろん海外の番号で、国際電話とかしたことない＆リスニング苦手な私はパニックになりかけましたが、「予備で持ってるMacBookならこのwebカメラより性能いいと思うからそっちのPCでセットアップしていい？」と試験管に伝え、MacBookから再度セットアップし事なきを得ました。(試験環境の撮影含めて初めからやり直しです泣)

性能が良いwebカメラを持っている方なら大丈夫かと思いますが、2つ以上ノートPC(webカメラ付)を持っている方は念のため持っていくことをオススメいたします。

## 4.試験開始

やっと事前準備が終わりました。
この時点で試験開始ボタンを押してから**約40分**かかってます。(バッファを考えたほうがいいというのはこのためです)
ですが試験時間はしっかり2時間確保されるのでご安心ください。

ここから試験に入ります。
よく言われている遅延について、私はそれほど気になりませんでした。
もちろん多少はありますが、Udemyで実施したkodekloudの操作くらいの遅延でしたので、そこで気にならなければ問題ないと思います。
一応「環境が重くなったら押してね」というリフレッシュボタンがありましたので、とんでもなく重くなってしまった場合はそこを押せば治るのかと思われます。

問題についてですが、私の場合は17問出題されました。
これはどの方も仰っている事ですが、少しでも解くのに時間がかかりそうだと感じたらフラグをつけて後回しにしましょう。
私の場合は、序盤難しめ中盤易しめ終盤普通といった構成でした。

ブラウザに関して、試験問題中に使用されるトピックのドキュメントリンク(kubernetes.io)があるのでそこからアクセスします。
問題ごとに押しているとタブがどんどん増えていきますが、見辛くなるので私は一度開いた後はサイト内検索で移動してました。
ブラウザの右上に三転リーダかハンバーガーメニュー(すみません忘れました)がありますのでそこから倍率を下げると見やすいですし、コピペもしやすいです。

問題文は基本的には英語で、読解に自信がない箇所だけ日本語に切り替えて実施しました。

制限時間が終了したら、画面が切り替わり試験管からチャットが来てアプリが閉じるのでそこで終了です。

# 試験結果

| 受験日    | 合格ライン | スコア | 結果 | 証明バッヂ                                                                              |
| --------- | ---------- | ------ | ---- | --------------------------------------------------------------------------------------- |
| 2023/8/31 | 66%        | 81     | pass | [Credly](https://www.credly.com/badges/262e277c-da5e-4456-a035-90376ca97dc0/public_url) |

# 受験後の感想

これまで経験した資格試験とは違いハンズオン形式ということで緊張しましたが、無事乗り切れて一安心です。

正直試験を始める前が一番大変で、トラブル時冷静に対処する能力が培われた気がします。
``英語でのコミュニケーション``+``カメラトラブル発生``で試験前にメンタルダメージをくらいましたが、落ち着いて対処し問題に臨めば十分に得点できる試験だと思います。
試験前に何をするのかあらかじめ知っておき、メンタル面での準備をしておくと良いでしょう。
冒頭にも書いていますが、この記事が皆様のお役に立てれば幸いです。

CKADとは試験内容もほとんどかぶっていて、新たにインプットする知識もそれほど多くなさそうなのでこのままCKADの取得も目指したいと思います。

この記事を開いていただき、
また、ここまで読んでいただきありがとうございました。
