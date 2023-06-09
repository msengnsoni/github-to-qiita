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

