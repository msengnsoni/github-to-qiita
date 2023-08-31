---
title: "【対策メモ】Certified Kubernetes Administrator (CKA)試験"
topics: ["Kubernetes", "CKA", "資格"]
published: true
---

# はじめに

本記事は、Certified Kubernetes Administrator (CKA)試験の対策をしている上で覚えておくべきだと感じたことをメモとして残しておくものです。

# Cluster Architecture

- Master Node
  - Kube-api Server
  - ETCD Cluster
  - Kube-Scheduler
  - Controller-Manager
    - Node-Controller
    - Replication-Controller
    - etc...

- Worker Node
  - kubelet
  - Kube-proxy
  - Container Runtime Engine (docker, rkt, containerd)

# Kube-Scheduler

- podがどのノードに配置されるかを**決定するだけ**
  - 実際にpodを配置する仕事はkubeletが実行する

# Pod

- 作成コマンド``kubectl run``
- 特定のnode上にpodを作成した場合、``spce.nodeName``フィールドに記述が簡単
- ``spec.containers.command``フィールド → pod立ち上げ時に実行するコマンドを設定
  - DockerfileのENTRYPOINTを上書き
- ``spec.containers.args``フィールド → ``spec.containers.command``に与える引数を設定
  - DockerfileのCMDを上書き

# Deployment

- 作成コマンド``kubectl create deployment``
  - ``create``コマンドで``--labels``はサポートしていないため注意

# Service

- 作成コマンド``kubectl expose {pod/rs/deploy}``
- DNS Aレコードは``my-svc.my-namespace.svc.cluster.local``の形式
- 各Portの概念
  - TargetPort = あて先pod側の待ち受けport
  - Port = Service自信の利用port
  - NodePort (type=NodePort利用時) = 外部からアクセスするための利用port
  - あくまで主観はserviceであることを意識する
- pod作成と同時にserviceを作成する場合は``--expose=true``を``kubectl run``コマンド実行時に指定

# TaintとToleration

- Taint作成コマンド``kubectl taint node``
- ノードが特定のpodを受け入れないように制限する
  - 特定のノードに配置されることを保証するものではないことに注意
- masterノードにpodが配置されないようになっているのもこの機能によるもの

# Affinity

- NodeAffinity
  - ラベルとの組み合わせでpodに対して特手のノードに配置されることを強制する
  - spec配下のフィールドに記載、記載要領は[公式サイト](https://kubernetes.io/ja/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)を要参照
- PodAffinity
  - 特定のPodが存在するNodeへスケジューリングする
- podAntiAffinity
  - 特定のPodが存在していないNodeへスケジューリングする

# Resource要求と制御

- podの使用するリソースを制御する方法を大きく3つ
  - 各podのmanifestファイル内(spec.containers.resources配下)に記述する
  - LimitRangeオブジェクトを作成する
    - podに対してリソース制限をかける
  - ResourceQuotaオブジェクトを作成する
    - namespaceに対してリソース制限をかける

# Static Pod

- デフォルトのパスは``/etc/kubernetes/manifests``
  - kubeconfig.yaml(kubeletの``--config``オプションに設定されているファイル)の``staticPodPath``がstatic podのmanifestファイルを配置するパスになる
  - kubeletの``--pod-manifest-path``オプションがmanifestファイルを配置するパスになる

# Scheduler Plugins

- スケジューラーフレームワーク
  - Kubernetesのスケジューラーに対してプラグイン可能なアーキテクチャ
  - [公式サイト](https://kubernetes.io/ja/docs/concepts/scheduling-eviction/scheduling-framework/)の記載を要参照

# configMap

- cm作成コマンド``kubectl create cm <name> --from-literal=<key>=<value>``

# Secret

- secret作成コマンド``kubectl create secret generic <name> --from-literal=<key>=<value>``

# initContainer

- アプリのコンテナが起動する前に実行される1つ以上のinitコンテナを持つことができる
- containersフィールドと同じインデントに``initContainers``フィールドを記載する

# 各componentのversion管理

- Kube-apiserver = Xの場合 (1.10の場合)
  - Controller-manager = X-1 (1.9 or 1.10)
  - Kube-scheduler = X-1 (1.9 or 1.10)
  - Kubelet = X-2 (1.8 or 1.9 or 1.10)
  - Kube-proxy = X-2 (1.8 or 1.9 or 1.10)
  - Kubectl = X+1 > X-1 (1.9 or 1.10 or 1.11)

# etcdのバックアップ/リストア

- 基本的には[公式](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)を参照
- etcdctlコマンドに渡すオプションに必要な情報はetcdのpodをdescribeして確認 or 動作nodeに入り``ps aux | grep etcd``でオプションを確認
- etcdが外部サーバの場合(etcdサーバに入り実行)
  - ``ps aux | grep etcd``でオプション確認
  - ``systemctl status etcd``で実行サービスのpath確認
- リストア時に``--data-dir``でdataディレクトリを変更した場合
  - etcdがpodの場合(動作nodeに入り実行)
    - ``/etc/kubernetes/manifest/etcd.yaml``内のvolume及びvolumeMountフィールドを変更
  - etcdが外部サーバの場合(etcdサーバに入り実行)
    - etcdのサービスファイルを編集しオプションの値を変更
    - ``--data-dir``で指定したディレクトリの権限を``chown -R``で変更、基本的に以前指定していたディレクトリと合わせる
    - ``systemctl daemon-reload``と``systemctl restart etcd``を実行しプロセス再起動

# Cluster Network

- クラスタに設定されているNetworking Solutionを確認したい
  - ``/etc/cni/net.d``ディレクトリを確認
- serviceのIP range(CIDR)を確認したい
  - api-serverのmanifestファイル内(デフォルトで``/etc/kubernetes/manifests/``配下)``--service-cluster-ip-range``オプションを確認

# Troubleshooting

- apiserverのデフォルトポートは``6443``
- k8sリソースの確認コマンドは``kubectl api-resources``
- systemd管理のユニットファイル内に変更を加えた後は``systemctl daemon-reload``を忘れない

# metrics server

- 導入後``kubectl top``が使用可能

# cluster update

- ツール(kubeadm,kubelet,kubectl)のアップグレード先versionを確認
  - ``apt show <tool_name> -a | grep <version_number>``コマンドを実行

# kubeadm

- worker nodeを新規にjoinさせるコマンドを出力
  - master nodeで``kubeadm token create --print-join-command``コマンドを実行
- 証明書の有効期限を出力
  - master nodeで``kubeadm certs check-expiration``コマンドを実行
- 証明書を更新
  - master nodeで``kubeadm certs renew <target_comportnent>``コマンドを実行

# 試験開始後のセットアップ

- **kubectl Cheat Sheet**の最初に記載のコマンドを実行(kubectlのオートコンプリート、k=kubectlに)
- ``export do="--dry-run=client -o yaml"``

# コマンド関連

- ``--record=true``をつけて実行すると``kubectl rollout``で状態や履歴の確認が可能
- k8s関連コマンド以外で使い方をわかっておいたほうが良いコマンド
  - ps
    - aux
  - ssh
  - openssl x509
    - -in
    - -text
  - journalctl
    - -u
  - systemctl (service)
    - daemon-reload
    - status
    - restart
    - start
    - stop
  - curl (wget)
  - nslookup
  - grep
    - -A
    - -B
    - -i
  - vim (vim操作)
