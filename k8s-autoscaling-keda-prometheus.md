<!--
title:   【k8s】EKS環境でKEDA + Prometheusを試してみた
tags:    keda,kubernetes,prometheus
id:      dca37a623986d21cd63f
private: true
-->
# 環境構成図

今回構成した図は以下になります。なお、Prometheusの詳細なコンポーネントは省略しています。
![KEDA構成.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/e3fd2b0d-29d7-a1eb-47d5-917f858bccc3.jpeg)


# 環境準備
## AWS認証情報を定義
```
AWS_ACCOUNT=************
AWS_DEFAULT_REGION=ap-northeast-1
AWS_ACCESS_KEY_ID=************
AWS_SECRET_ACCESS_KEY=************

echo export AWS_ACCOUNT=$AWS_ACCOUNT >> ~/.bash_profile
echo export AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION >> ~/.bash_profile
echo export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID >> ~/.bash_profile
echo export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY >> ~/.bash_profile
source ~/.bash_profile
```

## kubectlのインストール
```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.22.0/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ mkdir -p $HOME/bin && mv ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
```
## ekctlのインストール
```
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/0.61.0/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ sudo mv /tmp/eksctl /usr/local/bin
```

## helmのインストール
```
$ mkdir ~/environment/helm
$ cd ~/environment/helm/
$ curl -L https://git.io/get_helm.sh | bash -s -- --version v3.8.2
```
## kubernetes用リポジトリの追加
```
$ helm repo add stable https://charts.helm.sh/stable
$ helm repo update
```

## クラスタ定義ファイルの作成
<details>
<summary>cluster-config.yaml</summary><div>

```bash:cluster-config.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: work-cluster
  region: ap-northeast-1
  version: "1.21"
managedNodeGroups:
  - name: work-nodegroup
    ami: ami-02a49655b336f1b47
    overrideBootstrapCommand: |
      #!/bin/bash
      /etc/eks/bootstrap.sh work-cluster --container-runtime containerd
    instanceType: t2.medium
    volumeSize: 20
    desiredCapacity: 1
    minSize: 1
    maxSize: 3
```
</div></details>

## クラスタを作成
```
$ eksctl create cluster -f cluster-config.yaml
```

# nginxとnginx-exporterのデプロイ
## nginxのmanifestを定義(ConfigMap,Service,deployment)

<details>
<summary>nginx-manifest.yaml</summary><div>

```bash:nginx-manifest.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
  labels:
    app: nginx
data:
  nginx.conf: |2

    user  nginx;
    worker_processes  1;
    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;
    events {
        worker_connections  1024;
    }
    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
        access_log  /var/log/nginx/access.log  main;

        server {
            listen   8080;
            location /metrics {
                stub_status;
            }
        }

        sendfile        on;
        keepalive_timeout  65;
        include /etc/nginx/conf.d/*.conf;
    }
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - name: nginx-http
    protocol: TCP
    port: 80
    targetPort: 80
  - name: nginx-exporter
    protocol: TCP
    port: 9113
    targetPort: 9113
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.2
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-conf
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
      - name: nginx-exporter
        image: nginx/nginx-prometheus-exporter:0.8.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9113
        args:
          - -nginx.scrape-uri=http://localhost:8080/metrics
      volumes:
      - name: nginx-conf
        configMap:
          name: nginx-configmap
          items:
          - key: nginx.conf
            path: nginx.conf
```
</div></details>

### 躓きポイント
* Kind: Deploymentのspec.template.spec.containers.name: nginx-exporter部分(サイドカー)の記述
    * nginx-exporterを構築するために、exporter-podをapp-podのサイドカーとして登録する
    * argsの記述は、EKSでの構築だとしてもlocalhostのパスで動作を確認
* Kind: ConfigMapの記述
    * nginx-exporterを構築するために、nginx.confの記載を変更する必要があるため、manifest上でnginx.confを定義しConfigMapでnginx-pod内のnginx.confを上書きする

## nginxをデプロイ
```
$ kubectl apply -f nginx-manifest.yaml
```

# Prometheusのデプロイ
## Prometheus用のnamespaceを作成
```
$ kubectl create ns prometheus
```

## Prometheusのインストール
* 今回はprometheus-operatorを採用
```
$ helm install prometheus stable/prometheus-operator --namespace prometheus
```

## Prometheusのダッシュボードに接続確認 (Cloud9で構築した場合)
```
$ kubectl port-forward service/prometheus-prometheus-oper-prometheus 8080:9090 --address 0.0.0.0 -n prometheus
```
* Tools > Preview > Preview Running Application でブラウザを開く

## ServiceMonitorリソースの定義
<details>
<summary>prometheus-servicemonitor.yaml</summary><div>

```bash:prometheus-servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx-servicemonitor
  namespace: prometheus
  labels:
    app: nginx
    release: prometheus
spec:
  endpoints:
  - port: nginx-exporter
  namespaceSelector:
    matchNames:
    - default
  selector:
    matchLabels:
      app: nginx
```
</div></details>

:::note info
ServiceMonitorの記述方法は、`kubectl explain servicemonitor.spec`で確認
:::

## ServiceMonitorリソースのデプロイ
```
$ kubectl apply -f prometheus-servicemonitor.yaml -n prometheus
```

* Prometheusのダッシュボードから、nginx-podがTargetsに含まれていることを確認
    * この手順通りに実施した場合、***prometheus/nginx-servicemonitor/0**という名前で表示される

# KEDAのデプロイ
## KEDAをグラフリポジトリに追加
```
helm repo add kedacore https://kedacore.github.io/charts
```

## リポジトリをアップデート
```
helm repo update
```

## KEDA用のnamespaceを作成
```
kubectl create namespace keda
```

## KEDAをインストール
```
helm install keda kedacore/keda --namespace keda
```

## ScaledObjectリソースの定義
<details>
<summary>keda-scaledobject.yaml</summary><div>

```bash:keda-scaledobject.yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
 name: nginx-scale
 namespace: default
spec:
 scaleTargetRef:
   kind: Deployment
   name: nginx-deploy
 minReplicaCount: 1
 maxReplicaCount: 2
 cooldownPeriod: 30
 pollingInterval: 1
 triggers:
 - type: prometheus
   metadata:
     serverAddress: http://prometheus-operated.prometheus.svc.cluster.local:9090
     metricName: nginx_connections_active_keda
     query: |
       sum(nginx_connections_active{job="nginx-svc"})
     threshold: "2"
```
</div></details>

:::note info
分かりやすくするため、nginxサーバへのconnentionが`2`になったらスケーリングする設定にしています。
:::


### 躓きポイント
* spec.triggers.metadata.serverAddressの記述
    * prometheusのserviceをk8sのDNSで記述する
        * DNS名のルールは、`my-svc.my-namespace.svc.cluster.local`
    * prometheus-operatorでは、prometheus-operatedサービスが9090ポートで待ち受けている
* spec.triggers.metadata.queryの記述
    * スケーリングのトリガーにしたいクエリをPromQL表記で記述する
* spec.triggers.metadata.thresholdの記述
    * トリガーとなるspec.triggers.metadata.queryで指定したクエリの閾値を指定する
* spec.scaleTargetRef.nameの記述
    * スケール対象のdeploymentの名前を記述する
        * ScaledObjectリソースをデプロイするnamespaceと同じnamespace内のdeploymentを指定する必要がある

## ScaledObjectリソースのデプロイ
```
$ kubectl create -f keda-scaledobject.yaml
```




## HPAリソースの確認
```
$ kubectl get hpa
```

:::note info
ScaledObjectリソースが正常にデプロイされた場合、自動的にHPAリソースが作成される
:::



# スケールアウト動作確認
* nginxのブラウザで更新連打し一時的にconnectionを増加させます
```
$ kubectl describe hpa
Name:                                                                    keda-hpa-nginx-scale
Namespace:                                                               default
Labels:                                                                  app.kubernetes.io/managed-by=keda-operator
                                                                         app.kubernetes.io/name=keda-hpa-nginx-scale
                                                                         app.kubernetes.io/part-of=nginx-scale
                                                                         app.kubernetes.io/version=2.8.1
                                                                         scaledobject.keda.sh/name=nginx-scale
Annotations:                                                             <none>
CreationTimestamp:                                                       Wed, 02 Nov 2022 00:39:00 +0000
Reference:                                                               Deployment/nginx-deploy
Metrics:                                                                 ( current / target )
  "s0-prometheus-nginx_connections_active_keda" (target average value):  3 / 2
Min replicas:                                                            1
Max replicas:                                                            2
Deployment pods:                                                         1 current / 2 desired
Conditions:
  Type            Status  Reason              Message
  ----            ------  ------              -------
  AbleToScale     True    SucceededRescale    the HPA controller was able to update the target scale to 2
  ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from external metric s0-prometheus-nginx_connections_active_keda(&LabelSelector{MatchLabels:map[string]string{scaledobject.keda.sh/name: nginx-scale,},MatchExpressions:[]LabelSelectorRequirement{},})
  ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  ----                       -------
  Normal  SuccessfulRescale  9s    horizontal-pod-autoscaler  New size: 2; reason: external metric s0-prometheus-nginx_connections_active_keda(&LabelSelector{MatchLabels:map[string]string{scaledobject.keda.sh/name: nginx-scale,},MatchExpressions:[]LabelSelectorRequirement{},}) above target
```
```
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-69dfb9f967-2b7wp   2/2     Running   0          22s
nginx-deploy-69dfb9f967-f5f55   2/2     Running   0          4h51m
```

# スケールダウン動作確認
```
$ kubectl describe hpa
Name:                                                                    keda-hpa-nginx-scale
Namespace:                                                               default
Labels:                                                                  app.kubernetes.io/managed-by=keda-operator
                                                                         app.kubernetes.io/name=keda-hpa-nginx-scale
                                                                         app.kubernetes.io/part-of=nginx-scale
                                                                         app.kubernetes.io/version=2.8.1
                                                                         scaledobject.keda.sh/name=nginx-scale
Annotations:                                                             <none>
CreationTimestamp:                                                       Wed, 02 Nov 2022 00:39:00 +0000
Reference:                                                               Deployment/nginx-deploy
Metrics:                                                                 ( current / target )
  "s0-prometheus-nginx_connections_active_keda" (target average value):  1 / 2
Min replicas:                                                            1
Max replicas:                                                            2
Deployment pods:                                                         1 current / 1 desired
Conditions:
  Type            Status  Reason              Message
  ----            ------  ------              -------
  AbleToScale     True    ReadyForNewScale    recommended size matches current size
  ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from external metric s0-prometheus-nginx_connections_active_keda(&LabelSelector{MatchLabels:map[string]string{scaledobject.keda.sh/name: nginx-scale,},MatchExpressions:[]LabelSelectorRequirement{},})
  ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  ----                       -------
  Normal  SuccessfulRescale  56m   horizontal-pod-autoscaler  New size: 2; reason: external metric s0-prometheus-nginx_connections_active_keda(&LabelSelector{MatchLabels:map[string]string{scaledobject.keda.sh/name: nginx-scale,},MatchExpressions:[]LabelSelectorRequirement{},}) above target
  Normal  SuccessfulRescale  49m   horizontal-pod-autoscaler  New size: 1; reason: All metrics below target
m_satake:~/environment/download $
```
```
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-69dfb9f967-f5f55   2/2     Running   0          5h47m
```

以上