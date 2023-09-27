---
title: "SadServersの所感 (Easy編)"
topics: ["SadServers", "Linux"]
published: true
---

# 目次

- [目次](#目次)
- [本記事の概要](#本記事の概要)
- [各章毎の所感](#各章毎の所感)
  - [1. "Saint John": what is writing to this log file?](#1-saint-john-what-is-writing-to-this-log-file)
  - [2. "Saskatoon": counting IPs.](#2-saskatoon-counting-ips)
  - [3. "Santiago": Find the secret combination](#3-santiago-find-the-secret-combination)
- [総括](#総括)

# 本記事の概要

学習のために[SadServers](https://sadservers.com/)を実施した際の所感になります。
対象のレベルはEasyです。

:::note warn
本記事では回答の直接的な解説はしていません。ご了承ください。
:::

# 各章毎の所感

## 1. "Saint John": what is writing to this log file?

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Correct              | no                          | -                |

1問目ということもあり基礎知識で回答できた。
プロセス情報を出すコマンドとして、``lsof``や``fuser``を頭に入れておけるとなお良い。

## 2. "Saskatoon": counting IPs.

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Incorrenct           | no                          | -                |

``uniq``の仕様がわかっていなかったため不正解。
[uniqのman](https://linuxjm.osdn.jp/html/GNU_textutils/man1/uniq.1.html)より引用

> uniq に与える入力はソートされていなければならない。比較は連続した行の間でのみ行われる。

勉強になります。

## 3. "Santiago": Find the secret combination

| Correct or Incorrect | Did you refer to next clue? | (How many times) |
| -------------------- | --------------------------- | ---------------- |
| Incorrenct           | yes                         | 2                |

``uniq``はとにかくsortしてからじゃないと駄目ということをよく学べた。

# 総括

Easyなのに正答率1/3という苦い結果。
しかし、学ぶことはとても多いので引続きMediumに挑戦する。
