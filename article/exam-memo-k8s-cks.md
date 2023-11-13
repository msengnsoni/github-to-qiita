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
  - ``kube-bench``コマンドで実行
    - ``kube-bench run --targets=master``
      - maseterノードに対して実行
    - ``kube-bench run --targets=node``
      - workerノードに対して実行

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
  - ``netstat -plnt``にgrepで確認
    - ``-p``はPID/Programを表示
    - ``-l``はリスニング状態のソケットのみ表示
    - ``-n``はホスト名やサービス名を解決せずに、IPアドレスやポート番号を表示
    - ``-t``はTCPプロトコルのみ表示

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
    - ``-a (--add)``
    - ``-p (--preprocess)``
- podに設定したい場合
  - annotaionsに``container.apparmor.security.beta.kubernetes.io/<container_name>: <profile_ref>``を追加
    - ``<profile_ref>``は以下の3種類
      - runtime/default: ランタイムのデフォルトのプロファイルを適用する
      - localhost/<profile_name>: <profile_name>という名前でホストにロードされたプロファイルを適用する
      - unconfied: いかなるプロファイルもロードされないことを示す

# OPA Gatekeeper

- Kubernetesリソース（ポッド、サービス、ネームスペースなど）に適用されるポリシーを定義し、それらのポリシーを実行してリソースの作成や変更を制御する
- Validating Admission Controllerとして動作し、Kubernetes APIリクエストが送信される前にポリシーの適用を確認する
- ValidatingWebhookConfigrationリソース内でOPAサーバの情報を記載
- ``.rego``ファイルからconfigmapを作成する場合は``--from-file=``で指定
- ``ConstraintTemplate``リソース内にconstraintを強制するRegoロジックとコンストレイントのスキーマの両方が記述されているためまずここを確認
  - ``kubectl get crd``で各リソース名を確認

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
- 検出された脅威は``journalctl -u falco``で確認する

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
- DNSのみ通信許可したい場合のサンプルyaml
  - portsにTCP/UDPの53のみ許可の設定を記載する

<details><summary>サンプルyaml</summary>

```rb
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  - Ingress
  egress:
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
```
</details>

# apiserver

- オプション
  - ``--kubernetes-service-node-port``
    - apiserverのセットアップ時に作成されるserviceのタイプがnodePortになり、指定したport番号を使用する
    - このオプションが指定されていない場合、作成されるserviceのタイプは**clusterIP**になる
    - この項目を変更した際は、既存のservcieを削除しないと新たなタイプのserviceが作成されないため注意
- kube-apiserverが立ち上がってこない時にログを見て調査したい
  - ``/var/log/pods`` or ``/var/log/containers``
    - kube-apiserverの起動に失敗している場合は関連ログ(読み込めないフラグ等)が置かれている
  - ``journalctl -u kubelet``
  - ``crictl logs <kube-apiserverのCONTAINER_ID>``
  - ``journalctl | grep kube-apiserver``
  - ``cat /var/log/syslog | grep kube-apiserver``
- kubeletのconfigファイル(``/etc/kubernetes/kubelet.conf``等)をKUBECONFIG環境変数に設定することでapiserverと通信可能
  - NodeRestrictionを有効化することによって制限できる
- TLS接続における最小バージョンと暗号スイートの設定
  - ``--tls-min-version=VersionTLS12``
  - ``--tls-cipher-suites=TLS_AES_128_GCM_SHA256``
    - 値は問題で指定されるものでよい

# ETCD

- ETCDはデータを``/registry/{type}/{namespace}/{name}``配下に書き込む
- etcdctlで参照する際は``etcdctl (cert関連オプション) get``を使用する
- ``--cipher-suites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256``
  - 値は問題で指定されるものでよい
- api-serverの設定変更し起動してくることを確認 → ETCDの設定変更という順番でやらないとETCDが上がってこなかった(たまたま？)

# Pod Security Standards

- namespace単位でpodにセキュリティ制御をかける
- namespaceリソース内の``metadata.annotations``に設定する
- 基本は3つのprofileから選択
  - Privileged
    - 特権を許可(ほぼ制限なし)
  - Baseline
    - 既知の特権昇格を防止する最小限の制限ポリシー
  - Restricted
    - 厳しく制限されたポリシーで、現在のpod hardeningのベストプラクティスに従う

# Immutability for contaniners

- bash/shellの削除
  - startupProbeで``remove /bin/bash``
- filessytemをreadonlyに
  - securitycontextで設定
- root userでコンテナを実行しない(run as user and non root)
  - securitycontextで設定

# Secure and Herden Images

- imageに対してのベストプラクティス
  - パッケージバージョンを明確にすること(latestは使わない)
- root userで実行しない
- filessytemをreadonlyに
- shell accessを削除

# Other

- curlを用いてkubernetesリソースを取得したい
  - ひな形を覚えておく(↓認証にtokenを使う場合)
  - ``curl htts://kubernetes.default/api/v1/namespaces/default/pods -H "Authorization: Bearer $(token)" -k``

# Command

- kubectl
  - config
    - get-context -o name
      - context nameを表示する
    - view --raw
      - 機密データを表示する
- sha512sum
  - SHA-512ハッシュ値を計算して表示する
  - kubernetesコンポーネントファイルのバイナリを比較する際に使用
- strace
  - -p
    - 指定したPIDのプログラムがどのシステムコールを実行し、それに伴う引数や結果を表示する
  - -c
    - 統計情報として表示
- systemctl
  - list-units --type service
    - systemctlが管理しているunitの一覧を表示
