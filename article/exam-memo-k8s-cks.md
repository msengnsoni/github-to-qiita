---
title: "【対策メモ】Certified Kubernetes Security Specialist (CKS)試験"
topics: ["Kubernetes", "cks", "資格"]
published: false
---

# はじめに

本記事は、Certified Kubernetes Security Specialist (CKS)試験の対策をしている上で覚えておくべきだと感じたことをメモとして残しておくものです。

# Benchmard

- CIS Benchmark
  - CIS-CAT
    - CISが提供するCIS Benchmarksへの準拠をチェックするツール
    - kubernetes用のCIS-CATはPro版(有償)にしかないため注意
- kube-bench
  - CIS Benchmarksに基づくOSSのチェックツール

# Kubelet Security

- kubeletと通信するポート
  - **10250**
    - フルアクセス
    - kubelet.serviceの設定値``--anonymous-auth``はデフォルトで``true``のため``false``推奨
      - ``false``の場合は``curl``送信時にオプションでclient ca fileを渡せば認証される
  - **10255**
    - 読み取り専用
    - 無効にする場合は``readOnlyPort``を``0``に設定

# kubectl proxy と kubectl port-forward

- kubectl proxy
  - API serverへの通信をプロキシ
- kubectl port-forward
  - 展開されたpodと通信するためのポートフォワーディング

# Linux Hardening

- 特定のユーザをログイン不可(sshやsu不可)にしたい
  - ``usermod -i /sbin/nologin <username>``
- sshログイン時の挙動制御
  - ``/etc/sshd/sshd_config``を編集
- sudo実行可能ユーザの制御
  - ``/etc/sudoers``を編集
- ポート番号を私用しているプログラムが知りたい
  - ``netstat -antp``にgrepで確認
    - ``-p``はPID/Programを表示
- 
