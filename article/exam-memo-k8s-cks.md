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

# Seccomp

- プロセスのシステムコールを制限し、アプリケーションやプロセスのセキュリティを向上させる
- デフォルトのseccompプロファイルの場所
  - ``/var/lib/kubelet/seccomp``
- 特定のseccompプロファイルを指定してPodを作成する
  - ``spec.securityContext.seccompProfile.lolalhostProfile``にパスを設定
  - 記載するパスはkubeletの起動引数で指定されたseccompプロファイルの場所以下からで良い(デフォルトでは``/var/lib/kubelet/seccomp/``)

# AppArmor

- 特定のプロセスが実行できる操作やアクセスできるファイルなどのシステムリソースに対するアプリケーションのアクセス制御を実施する
- 関連コマンド
  - ``aa-status``
  - ``apparmor-parser``
    - ``-a``
    - ``-p``
- podに設定したい場合
  - annotaionsに``container.apparmor.security.beta.kubernetes.io/<container_name>: <profile_ref>``を追加
    - ``<profile_ref>``は以下の3種類
      - runtime/default: ランタイムのデフォルトのプロファイルを適用する
      - localhost/<profile_name>: <profile_name>という名前でホストにロードされたプロファイルを適用する
      - unconfied: いかなるプロファイルもロードされないことを示す

# OPA in kubernetes

- ValidatingWebhookConfigrationリソース内でOPAサーバの情報を記載
- ``.rego``ファイルからconfigmapを作成する場合は``--from-file=``で指定

# Container runtime

- runc
  - kubernetesデフォルトのcontainer runtime
- gVisor
  - ホストのオペレーティングシステムとコンテナ化されたアプリケーションの間に中間層として機能
  - docker実行時にオプションで指定可能
    - ``--runtime runsc``
- kata container
  - 仮想マシンを作成し、その中でコンテナを実行
  - docker実行時にオプションで指定可能
    - ``--runtime kata``

# RuntimeClass

- ``RuntimeClass``リソースを作成後、Podの``spec.runtimeClassName``にリソース名を記載

# Pod間のmTLS通信

- 一般的にIstioが使用

# Admission Controller

- admission controllerの有効化
  - kube-apiserverの起動オプションに``--enable-admission-plugins``を追記して指定
- admission controller用のconfigファイルを明示的に指定
  - kube-apiserverの起動オプションに``--admission-control-config-file``を追記して指定

# Kubesec

- podのスキャニングを提供
- ``kubesec``コマンドを使用、詳細な使い方は``-h``で確認

# Trivy

- コンテナイメージの脆弱性スキャンを提供
- ``trivy``コマンドを使用、詳細な使い方は``-h``で確認

# Falco

- Kubernetesクラスタ内で実行されるアプリケーションやサービスの動作を監視し、異常なアクティビティやセキュリティ違反を検出する
- デフォルトの設定ファイル
  - ``/etc/falco/falco.yaml``
- ルール設定ファイル
  - ``/etc/falco/falco_rule.yaml``
  - ``/etc/falco/falco_rule.local.yaml``
    - 記載することでデフォルトのルールをオーバーライド可能
- falcoのホットリロード
  - ``kill -1 $(cat /var/run/falco.pid)``
- outputのフォーマットを変えたい
  - outputフィールドを[公式ドキュメント](https://falco.org/docs/reference/rules/supported-fields/)を参考に変更する

# Audit

- kube-apiserverにフラグを追加することで有効化
  - ``--audit-policy-file``
    - policyリソース定義ファイルを指定
  - ``--audit-log-path``
    - auditログを書き込むファイルを指定
  - ``--audit-log-maxage``
    - 監査ログファイルを保持する最大日数を指定
  - ``--audit-log-maxsize``
    - ログファイルがローテーションされるまでの最大サイズをメガバイト単位で指定
  - ``--audit-log-maxbackup``
    - 保持する監査ログファイルの最大数を指定
- policyリソースの書き方に関する注意点
  - 基本的な書き方は公式ドキュメント参照
  - groupの記載が必要なリソースを対象にする場合は``kubectl api-resources``で確認すること

# Secret

- ServiceAccountTokenの自動マウントを止めたい(下記2つのうちいずれかの方法で実施)
  - podのマニフェストファイル内で``spec.automountServiceAccountToken``を``false``に設定
  - serviceaccountのマニフェストファイル内で``automountServiceAccountToken``を``false``に設定

# NetworkPolicy

- リソースにlabelが複数設定されている場合はその分も設定すること
  - kubernetes側の処理によって自動的に付与されるlabelは設定しなくても良い
- from配下の条件はAND条件になる
- from間の条件はOR条件になる
