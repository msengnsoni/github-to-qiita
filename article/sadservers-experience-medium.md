---
title: "SadServersの所感 (Medium編)"
topics: ["SadServers", "Linux"]
published: true
---

# 目次

- [目次](#目次)
- [本記事の概要](#本記事の概要)
- [各章事毎の所感](#各章事毎の所感)
  - [4. "Saint John": what is writing to this log file?](#4-saint-john-what-is-writing-to-this-log-file)
  - [5. "Saskatoon": counting IPs.](#5-saskatoon-counting-ips)
  - [6. "Cape Town": Borked Nginx](#6-cape-town-borked-nginx)
  - [7. "Salta": Docker container won't start.](#7-salta-docker-container-wont-start)
  - [8. "Venice": Am I in a container?](#8-venice-am-i-in-a-container)
  - [9. "Oaxaca": Close an Open File](#9-oaxaca-close-an-open-file)
  - [10. "Melbourne": WSGI with Gunicorn](#10-melbourne-wsgi-with-gunicorn)
  - [11. "Lisbon": etcd SSL cert troubles](#11-lisbon-etcd-ssl-cert-troubles)
- [総括](#総括)

# 本記事の概要

学習のために[SadServers](https://sadservers.com/)を実施した際の所感になります。
対象のレベルはMediumです。
サイトへの登録が必要な``12. "Kihei": Surely Not Another Disk Space Scenario``は実施していません。

:::note warn
本記事では回答の直接的な解説はしていません。ご了承ください。
:::

# 各章事毎の所感

## 4. "Saint John": what is writing to this log file?

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Incorrect            | no                          | -                |

アプリ(db)のlogなのになぜか/var/log/messagesに出力されると勘違い。
systemdの``active (existed)``がどういう状態なのかも勉強になった。

## 5. "Saskatoon": counting IPs.

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Correct              | yes                         | 2                |

iptablesの操作を忘れていた。LPICを勉強していたころを思い出して懐かしい感覚。

## 6. "Cape Town": Borked Nginx

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Incorrect            | no                          | -                |

途中まではあっていたが最後のアプローチが間違っていた。
``nginx.service``にも関連設定値があると思わず、ずっと``nginx.conf``内の設定が原因だと思い込んで格闘していた。
普通にエラーログでググったら解決策が判明したので悔しい。

## 7. "Salta": Docker container won't start.

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Incorrenct           | no                          | -                |

時間切れにより失敗。
``lsof``が入っておらず、8888ポートをつかんでいるプロセスを探すのに時間がかかった。
後から``ss``or``netstat``でも確認できると知り試したが、なぜかプロセス欄が空欄で結局プロセスは分からない仕様？
解説記事を読み、``curl``に``-v``オプション付けて探るのか～と納得。勉強になります。

## 8. "Venice": Am I in a container?

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| -                    | -                         | -                |

``.dockerenv``がないしVM！とか思って解説記事読んだら撃沈。（そもそも問題文は**コンテナ**かVMかなのでdockerとは限らない）
ルートディレクトリに明確なi-nodeが振られているなんて全く知らなかった。
i-nodeという言葉を見たとき「何それ？」なんて思ってしまって調べたらLPIC101の範囲...
日々勉強です。

## 9. "Oaxaca": Close an Open File

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Incorrenct           | -                           | -                |

ファイルディスクリプタ(FD)についての理解が不足していた。
こちらの記事が大変参考になった。

- [シェルとファイルデスクリプタのお話](https://qiita.com/ueokande/items/c75de7c9df2bcceda7a9)

## 10. "Melbourne": WSGI with Gunicorn

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Correnct             | yes                         | 3                |

ヒントを三回見てなんとか正解。
gunicorn関連の設定をどうやって探すのか疑問だったが、解説記事を読んで納得。
``grep``をもっと使いこなしたい。

## 11. "Lisbon": etcd SSL cert troubles

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Correct              | yes                         | 2                |

``date``でホストの時間設定変えるとかアリなのかな？トラブル解決後に戻すならアリなのか...
iptablesの復習に最適。こちらのブログがとても分かりやすく参考になった。

- [はてなブログ-Carpe Diem](https://christina04.hatenablog.com/entry/iptables-outline)

# 総括

ヒントを見ればなんとか分かる問題がちらほらとありましたが、トラブルシューティングのアプローチについては省かれています。
Zennでアプローチまで含めて解説記事を書いてくださる方がいるので、正答したとしてもそこを見つつ要復習ですね。
もはや正解不正解はあまり関係ないですが、一応このフォーマットのままHard編も書いていこうと思います。
