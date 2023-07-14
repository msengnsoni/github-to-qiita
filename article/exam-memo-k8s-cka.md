---
title: "【対策メモ】Certified Kubernetes Administrator (CKA)試験"
topics: ["Kubernetes", "CKA", "資格"]
published: false
---

# はじめに

本記事は、Certified Kubernetes Administrator (CKA)試験の対策をしている上で覚えておくべきだと感じたことを
メモとして残しておくものです。

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

# Deployment

- 作成コマンド``kubectl create deployment``

# Service

- 作成コマンド``kubectl expose pod``
- DNS Aレコードは``my-svc.my-namespace.svc.cluster.local``の形式
- NodePort
  - TargetPort = あて先pod側の待ち受けport
  - Port = Service自信の利用port
  - NodePort = 外部からアクセスするための利用port
  - あくまで主観はserviceであることを意識する
- pod作成と同時にserviceを作成する場合は``--expose=true``を``run``コマンド実行時に指定

# TaintとToleration

- Taint作成コマンド``kubectl taint node``
- ノードが特定のpodを受け入れないように制限する
  - 特定のノードに配置されることを保証するものではないことに注意
- masterノードにpodが配置されないようになっているのもこの機能によるもの

# Node Affinity

- ラベルとの組み合わせでpodに対して特手のノードに配置されることを強制する
- spec配下のフィールドに記載、記載要領は[公式サイト](https://kubernetes.io/ja/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)を要参照

# Resource要求と制御

- podの使用するリソースを制御する方法を大きく3つ
  - 各podのmanifestファイル内(spec.containers.resources配下)に記述する
  - LimitRangeオブジェクトを作成する
    - podに対してリソース制限をかける
  - ResourceQuotaオブジェクトを作成する
    - namespaceに対してリソース制限をかける

# Static Pod

- デフォルトのパスは``/etc/kubernetes/manifests``
- kubeconfig.yamlの``staticPodPath``がstatic podのmanifestファイルを配置するパスになる

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