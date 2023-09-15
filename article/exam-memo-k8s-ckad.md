---
title: "【対策メモ】Certified Kubernetes Application Developer (CKAD)試験"
topics: ["Kubernetes", "ckad", "資格"]
published: true
---

# はじめに

本記事は、Certified Kubernetes Application Developer (CKAD)試験の対策をしている上で覚えておくべきだと感じたことをメモとして残しておくものです。
なお、CKA取得している知識を前提としたメモになります。ご了承ください。
CKA取得時の勉強メモは[こちら](https:qiita.com/msengnsoni/items/be6ad6e81f9e36414935)

# Readiness Probe and Liveness Probe

- readinessProbe
  - コンテナが**準備できているか**のチェック
- livenessProbe
  - コンテナが**生存しているか**のチェック

# Jobs and CronJobs

- Job
  - ``spec.completions``
    - 設定した回数分ジョブを完了するまでpodを作成し続ける
  - ``spec.parallelism``
    - 設定した数分podで並列処理する
- CronJob
  - ``spec.jobTemplate``配下にJobで設定する``spec``配下を配置
  - CronJob内で``spec``が3つになるので注意
  - Jobでも設定できる項目はCronJobでも設定可能
- 両リソースともに``kubectl create``で作成可能

# Stateful Sets

- なぜStateful Setを使うのか？
  - Deploymentでは各Podにランダムな数字が付与されるが、Stateful Setでは``Pod名-0````Pod名-1``のように序数が割り振られる。
  - これにより通信に特定の名前が必要なワークロードや、特定の順番で立ち上がる必要があるワークロードを実現できる。
- PersistentVolumeClaimを``spec.template.spec.volumes``を通常通り設定すると、各Podは同一のPVを割り当てる
  - sts内のPod間でvolumesを共有したくない場合、``spec.volumeClaimTemplates``にPVCの``metadata.*``と``spec.*``の記載が必要

# Headless Service

- 特手のPodに対してサブドメインとして機能するservice
  - サブドメインの命名規則は``podname.headless-servicename.namespace.svc.cluseter-domain.example``
- Stateful Sets(sts)にHeadless Serviceを設定した場合
  - stsは各podに固定の序数を付けるため、他リソースからsts内の特定のpodに対して通信が可能

| Headless Service名 | Headless Service<br>対象リソース | Podのhostname | 機能するサブドメイン名一覧                                                                                           |
| ------------------ | -------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------- |
| hoge               | Pod                              | pod           | pod.hoge.default.svc.cluster.local                                                                                   |
| hoge               | Devployment<br>(replicas=2)      | deploy        | deploy.hoge.default.svc.cluster.local<br>deploy.hoge.default.svc.cluster.local<br>(2つのPodが同じサブドメインをもつ) |
| hoge               | StatefulSet<br>(replicas=2)      | sts           | sts-0.hoge.default.svc.cluster.local<br>sts-1.hoge.default.svc.cluster.local                                         |

# Admission Controllers

- apiserverのリクエスト制御機能
  - apiserverの引数``--enable-admission-plugins``で設定される
  - 無効化したいプラグインは``--disable-admission-plugins``で設定
- apiリクエストを認証→認可した後のフェーズでリクエストを許容するか決める
- ``変異型(Mutating)``と``検証型(Validating)``に分けられる
  - 一般的に変異型→検証型の順番で検証される
- admission controllerはwebhookとして外部のものを利用可能
  - k8sクラスタ内にwebhook用podを作成して利用したり、外部のwebhookサーバを利用する
  - ``MutatingAdmissionWebhook``は順次処理
  - ``ValidatingAdmissionWebhook``は並列処理

# API

- クラスタ内でサポートされるapiバージョンを表示
  - ``kubectl api-versions``
- リソースが属するGROUP、KIND、VERSIONを表示
  - ``kubectl explain <resource>``
    - ``v1``とだけ表示されるリソースがあるがこれは**core**というGROUPに属しており省略可能なため書かれていない
- パスの構成
  - クラスタースコープの場合
    - ``/apis/GROUP/VERSION/*``
  - 名前空間スコープの場合
    - ``/apis/GROUP/VERSION/namespaces/NAMESPACE/*``
  - **core**に属するグループの場合は``/api``
- 各階層毎の詳細
  - ``/apis`` or ``/api``
    - 固定
  - ``/GROUP``
    - /appsとか/storage.k8s.ioとか
    - ``kubectl api-versions``で出てくるAPIVERSIONヘッダー部参照
  - ``/VERSION``
    - /v1とか/v1beta3とか
    - ``kubectl api-versions``で出てくるAPIVERSIONヘッダー部参照

```
// example
/api/v1/namespaces
/api/v1/pods
/api/v1/namespaces/my-namespace/pods
/apis/apps/v1/deployments
/apis/apps/v1/namespaces/my-namespace/deployments
/apis/apps/v1/namespaces/my-namespace/deployments/my-deployment
```

- apiリクエストを投げてbodyを見たい
  - ``kubectl proxy &`` → ``curl http://localhost:8001/<apipath>``で確認
    - & はバックグラウンドで実行

# Custom Resource Definition (CRD)

- 前提：リソースとコントローラーは互いに1対1の関係
  - 独自にリソースを定義できる
    - 対応するCustom Controllerがないと実際には何も影響を及ぼさない点に注意

# Deployment Strategy

- Blue Green
  - k8s公式では用意されていないため手動で設定
  - 例
    - Blue-Green環境それぞれにラベル:version=v1、version=v2を設定
    - serviceのselectorからversionを指定し向き先を変更
- Canary
  - k8s公式では用意されていないため手動で設定
  - 例
    - 両環境に同じラベルを設定
    - serviceは両環境に設定した同じラベルを指定
    - primary環境のreplicas=5の場合はcanary環境にreplicas=1の新versionのdeploymentをデプロイ
    - 問題がなければ徐々にcanary環境のreplicasを増やしprimary環境のreplicasを減らす
  - Istioを使えば詳細にルーティングの重みづけが可能
