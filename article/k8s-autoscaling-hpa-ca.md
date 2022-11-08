---
title: "eksを利用したk8sのオートスケーリングをやってみた(pod、node)"
topics: ["eks", "kubernetes"]
published: false
---

# Podのオートスケール
* Horizontal Pod Autoscaler (HPA)
  * podの水平スケーリングをCPU使用率に基づいて自動で実行する

kubectlのインストール
```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.22.0/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ mkdir -p $HOME/bin && mv ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
```
ekctlのインストール
```
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/0.61.0/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ sudo mv /tmp/eksctl /usr/local/bin
```

クラスタ定義ファイルの作成
```bash:cluster-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: eksworkshop-hpa
  region: ap-northeast-1
  version: "1.21"
managedNodeGroups:
  - name: eksworkshop-nodegroup
    ami: ami-02a49655b336f1b47
    overrideBootstrapCommand: |
      #!/bin/bash
      /etc/eks/bootstrap.sh eksworkshop-hpa --container-runtime containerd
    instanceType: t3.medium
    volumeSize: 20
    desiredCapacity: 3
    minSize: 1
    maxSize: 5
```

クラスタを作成
```
$ eksctl create cluster -f cluster-config.yaml
```

サンプルappのデプロイ
```
$ kubectl apply -f https://k8s.io/examples/application/php-apache.yaml
deployment.apps/php-apache created
service/php-apache created
```

HPAのデプロイ
```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```


metrics-serverの導入
このままget hpaしてもTARGETSがUnknownなのでmetrics-serverを導入
```
$ kubectl get hpa
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   <unknown>/50%   1         10        1          4h34m
```

metrics-serverの導入
```
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

:::note info
**memo** metrics-serverの導入により**kubectl top**が使用可能に
:::

TARGETSが0%表示になっていることを確認
```
$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          4h36m
```

別ターミナルで実行し負荷をかける
```
$ kubectl run -i \
    --tty load-generator \
    --rm --image=busybox \
    --restart=Never \
    -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

TARGETSに負荷がかかっていることを確認
```
$ kubectl get hpa
NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   250%/50%   1         10        6          6h37m
```
podが水平スケーリングされていることを確認
```
$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
load-generator                1/1     Running   0          2m4s
php-apache-779cd44bdc-hsx4h   1/1     Running   0          96s
php-apache-779cd44bdc-pnbvs   1/1     Running   0          111s
php-apache-779cd44bdc-qv24m   1/1     Running   0          96s
php-apache-779cd44bdc-shvtg   1/1     Running   0          51s
php-apache-779cd44bdc-t5s8f   1/1     Running   0          81s
php-apache-779cd44bdc-x4ptz   1/1     Running   0          7h49m
```

負荷を停止(CTRL+C)し、TARGETSを確認
```
$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          6h40m
```
podがスケールインされていることを確認(数分かかりました)
```
$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
load-generator                0/1     Error     0          8m26s
php-apache-779cd44bdc-x4ptz   1/1     Running   0          7h55m
```

phpリソースの削除
```
$ kubectl delete deployment.apps/php-apache service/php-apache horizontalpodautoscaler.autoscaling/php-apache
```

クラスタの削除
```
$ cd ~/environment
$ eksctl delete cluster -f cluster-config.yaml --wait
```

## マニフェストファイルでのHPAリソース作成
サンプルマニフェストファイル
```bash:sample-hpa.json
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60 # デフォルトは300(5分)
      policies:
        - type: Percent
          value: 20
          periodSeconds: 30
`
* spec配下がオートスケール設定
* maxReplicas = 最大レプリカ数
* minReplicas = 最小レプリカ数
* scaleTargetRefでオートスケール対象のリソースを指定
* metricsで閾値を指定
  * type : Resource = Podのリソース(CPU使用率)を対象にする
  * 他のカスタムメトリクスは[こちら](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#configurable-scaling-behavior)を参照
* behaviorでメトリクス収集間隔やスケール動作を指定
  * 詳細は[こちら](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#configurable-scaling-behavior)を参照


# Nodeのオートスケール
* Cluster Autoscaler (CA)
  * nodeの水平スケーリングをnodeがpendingになったことをトリガーに実行する

クラスタ定義ファイルの作成
`bash:cluster-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: eksworkshop-eksctl
  region: ap-northeast-1
  version: "1.21"
managedNodeGroups:
  - name: eksworkshop-nodegroup
    ami: ami-02a49655b336f1b47
    overrideBootstrapCommand: |
      #!/bin/bash
      /etc/eks/bootstrap.sh eksworkshop-eksctl --container-runtime containerd
    instanceType: t3.medium
    volumeSize: 20
    desiredCapacity: 3
    minSize: 1
    maxSize: 5
```

クラスタを作成
```
$ eksctl create cluster -f cluster-config.yaml
```

ASG更新
```
$ aws autoscaling \
    describe-auto-scaling-groups \
    --query "AutoScalingGroups[? Tags[? (Key=='eks:cluster-name') && Value=='eksworkshop-eksctl']].[AutoScalingGroupName, MinSize, MaxSize,DesiredCapacity]" \
    --output table

```
```
-----------------------------------------------------------------------------------
|                            DescribeAutoScalingGroups                            |
+------------------------------------------------------------------+----+----+----+
|  eks-eksworkshop-nodegroup-20c1e75b-20f3-9681-9315-f3c70949475e  |  2 |  8 |  3 |
+------------------------------------------------------------------+----+----+----+
```

```
# ASGの名前を取得
$ export ASG_NAME=$(aws autoscaling describe-auto-scaling-groups --query "AutoScalingGroups[? Tags[? (Key=='eks:cluster-name') && Value=='eksworkshop-eksctl']].AutoScalingGroupName" --output text)

# max capacityを4に増加
$ aws autoscaling \
    update-auto-scaling-group \
    --auto-scaling-group-name ${ASG_NAME} \
    --min-size 3 \
    --desired-capacity 3 \
    --max-size 4

# 変更を確認
$ aws autoscaling \
    describe-auto-scaling-groups \
    --query "AutoScalingGroups[? Tags[? (Key=='eks:cluster-name') && Value=='eksworkshop-eksctl']].[AutoScalingGroupName, MinSize, MaxSize,DesiredCapacity]" \
    --output table
```
```
-----------------------------------------------------------------------------------
|                            DescribeAutoScalingGroups                            |
+------------------------------------------------------------------+----+----+----+
|  eks-eksworkshop-nodegroup-20c1e75b-20f3-9681-9315-f3c70949475e  |  3 |  4 |  3 |
+------------------------------------------------------------------+----+----+----+
```

サービスアカウントを有効化
```
$ eksctl utils associate-iam-oidc-provider \
    --cluster eksworkshop-eksctl \
    --approve
```

Policyの作成
```
$ mkdir ~/environment/cluster-autoscaler

$ at <<EoF > ~/environment/cluster-autoscaler/k8s-asg-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup",
                "ec2:DescribeLaunchTemplateVersions"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
EoF

$ aws iam create-policy   \
  --policy-name k8s-asg-policy \
  --policy-document file://~/environment/cluster-autoscaler/k8s-asg-policy.json
```

kube-systemネームスペースにサービスアカウント(IAM role)を作成

```
$ eksctl create iamserviceaccount \
    --name cluster-autoscaler \
    --namespace kube-system \
    --cluster eksworkshop-eksctl \
    --attach-policy-arn "arn:aws:iam::${AWS_ACCOUNT}:policy/k8s-asg-policy" \
    --approve \
    --override-existing-serviceaccounts
```
サービスアカウントを確認
```
$ kubectl -n kube-system describe sa cluster-autoscaler
```
```
Name:                cluster-autoscaler
Namespace:           kube-system
Labels:              app.kubernetes.io/managed-by=eksctl
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::405501939914:role/eksctl-eksworkshop-eksctl-addon-iamserviceac-Role1-1ET8NSTU1N93S
Image pull secrets:  <none>
Mountable secrets:   cluster-autoscaler-token-7cknj
Tokens:              cluster-autoscaler-token-7cknj
```

CAのデプロイ
```
 $ kubectl apply -f https://www.eksworkshop.com/beginner/080_scaling/deploy_ca.files/cluster-autoscaler-autodiscover.yaml
clusterrole.rbac.authorization.k8s.io/cluster-autoscaler created
role.rbac.authorization.k8s.io/cluster-autoscaler created
clusterrolebinding.rbac.authorization.k8s.io/cluster-autoscaler created
rolebinding.rbac.authorization.k8s.io/cluster-autoscaler created
deployment.apps/cluster-autoscaler created
```

CA によって独自のポッドが実行されているノードを削除できないよう設定を追加
```
$ kubectl -n kube-system \
>     annotate deployment.apps/cluster-autoscaler \
>     cluster-autoscaler.kubernetes.io/safe-to-evict="false"
deployment.apps/cluster-autoscaler annotated
```

autoscaler imageをアップデート
```
#EKSバージョンで利用可能な最新のドッカーイメージを取得する必要があります
export K8S_VERSION=$(kubectl version --short | grep 'Server Version:' | sed 's/[^0-9.]*\([0-9.]*\).*/\1/' | cut -d. -f1,2)
export AUTOSCALER_VERSION=$(curl -s "https://api.github.com/repos/kubernetes/autoscaler/releases" | grep '"tag_name":' | sed -s 's/.*-\([0-9][0-9\.]*\).*/\1/' | grep -m1 ${K8S_VERSION})

$ kubectl -n kube-system \
>     set image deployment.apps/cluster-autoscaler \
>     cluster-autoscaler=us.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler:v${AUTOSCALER_VERSION}
deployment.apps/cluster-autoscaler image updated
```

cluster-autoscalerのログが見れることを確認
```
kubectl -n kube-system logs -f deployment/cluster-autoscaler
```

サンプルnginxアプリケーションをデプロイ
```
cat <<EoF> ~/environment/cluster-autoscaler/nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-to-scaleout
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        service: nginx
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx-to-scaleout
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 500m
            memory: 512Mi
EoF
```
:::note info
100m = 0.1コア、100Mi = 100MB
:::
```
$ kubectl apply -f ~/environment/cluster-autoscaler/nginx.yaml
deployment.apps/nginx-to-scaleout created
```

```
$ kubectl get deployment/nginx-to-scaleout
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
nginx-to-scaleout   1/1     1            1           3m4s
```

(スケールアウト実行前のノード状況)
```
$ kubectl get nodes
NAME                                                STATUS   ROLES    AGE     VERSION
ip-192-168-16-99.ap-northeast-1.compute.internal    Ready    <none>   4h57m   v1.21.2-13+d2965f0db10712
ip-192-168-63-234.ap-northeast-1.compute.internal   Ready    <none>   4h57m   v1.21.2-13+d2965f0db10712
ip-192-168-72-126.ap-northeast-1.compute.internal   Ready    <none>   4h57m   v1.21.2-13+d2965f0db10712
```

スケールアウト実行
```
$ kubectl scale --replicas=10 deployment/nginx-to-scaleout
deployment.apps/nginx-to-scaleout scaled
```

```
$ kubectl get pods -l app=nginx -o wide --watch
NAME                                 READY   STATUS              RESTARTS   AGE   IP               NODE                                                NOMINATED NODE   READINESS GATES
nginx-to-scaleout-6fcd49fb84-4h7zg   1/1     Running             0          17s   192.168.51.137   ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-7zwj9   0/1     ContainerCreating   0          17s   <none>           ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-9lvbr   1/1     Running             0          11m   192.168.4.237    ip-192-168-16-99.ap-northeast-1.compute.internal    <none>           <none>
nginx-to-scaleout-6fcd49fb84-gq29j   0/1     Pending             0          17s   <none>           <none>                                              <none>           <none>
nginx-to-scaleout-6fcd49fb84-jmzgh   1/1     Running             0          17s   192.168.31.40    ip-192-168-16-99.ap-northeast-1.compute.internal    <none>           <none>
nginx-to-scaleout-6fcd49fb84-jtktw   1/1     Running             0          18s   192.168.17.14    ip-192-168-16-99.ap-northeast-1.compute.internal    <none>           <none>
nginx-to-scaleout-6fcd49fb84-lvzf6   0/1     ContainerCreating   0          17s   <none>           ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-r9n6g   1/1     Running             0          18s   192.168.57.245   ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-tbtgg   1/1     Running             0          17s   192.168.38.131   ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-vfblw   0/1     ContainerCreating   0          18s   <none>           ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-7zwj9   1/1     Running             0          20s   192.168.69.206   ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-lvzf6   1/1     Running             0          20s   192.168.91.105   ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-vfblw   1/1     Running             0          21s   192.168.89.22    ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
```

nodeが追加されていることを確認
```
$ kubectl get nodes
NAME                                                STATUS   ROLES    AGE     VERSION
ip-192-168-16-99.ap-northeast-1.compute.internal    Ready    <none>   5h26m   v1.21.2-13+d2965f0db10712
ip-192-168-63-234.ap-northeast-1.compute.internal   Ready    <none>   5h26m   v1.21.2-13+d2965f0db10712
ip-192-168-72-126.ap-northeast-1.compute.internal   Ready    <none>   5h26m   v1.21.2-13+d2965f0db10712
ip-192-168-87-159.ap-northeast-1.compute.internal   Ready    <none>   119s    v1.21.2-13+d2965f0db10712
```

## スケールインの調査

replicasetを10→1に変更してみる
```
$ kubectl scale --replicas=1 deployment/nginx-to-scaleout
deployment.apps/nginx-to-scaleout scaled
```

podの状態を確認
```
$ kubectl get pods -l app=nginx -o wide --watch
NAME                                 READY   STATUS        RESTARTS   AGE    IP               NODE                                                NOMINATED NODE   READINESS GATES
nginx-to-scaleout-6fcd49fb84-2frlg   0/1     Terminating   0          9m5s   192.168.38.131   ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-4wx9w   1/1     Running       0          9m5s   192.168.71.48    ip-192-168-87-159.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-77tb8   0/1     Terminating   0          9m5s   192.168.86.226   ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-9lvbr   0/1     Terminating   0          38m    192.168.4.237    ip-192-168-16-99.ap-northeast-1.compute.internal    <none>           <none>
nginx-to-scaleout-6fcd49fb84-cf7cn   0/1     Terminating   0          9m5s   192.168.17.14    ip-192-168-16-99.ap-northeast-1.compute.internal    <none>           <none>
nginx-to-scaleout-6fcd49fb84-l6xvp   0/1     Terminating   0          9m5s   192.168.54.39    ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-lzjjd   0/1     Terminating   0          9m5s   192.168.16.207   ip-192-168-16-99.ap-northeast-1.compute.internal    <none>           <none>
nginx-to-scaleout-6fcd49fb84-xm6pw   0/1     Terminating   0          9m5s   192.168.73.122   ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-zwbk7   0/1     Terminating   0          9m5s   192.168.57.245   ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-l6xvp   0/1     Terminating   0          9m7s   192.168.54.39    ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-l6xvp   0/1     Terminating   0          9m7s   192.168.54.39    ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-2frlg   0/1     Terminating   0          9m7s   192.168.38.131   ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-2frlg   0/1     Terminating   0          9m7s   192.168.38.131   ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-zwbk7   0/1     Terminating   0          9m7s   192.168.57.245   ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-zwbk7   0/1     Terminating   0          9m7s   192.168.57.245   ip-192-168-63-234.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-77tb8   0/1     Terminating   0          9m8s   192.168.86.226   ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-77tb8   0/1     Terminating   0          9m8s   192.168.86.226   ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-xm6pw   0/1     Terminating   0          9m8s   192.168.73.122   ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
nginx-to-scaleout-6fcd49fb84-xm6pw   0/1     Terminating   0          9m8s   192.168.73.122   ip-192-168-72-126.ap-northeast-1.compute.internal   <none>           <none>
```

リソース供給過多のように思えるがスケールインされない様子...
```
$ kubectl get nodes
NAME                                                STATUS   ROLES    AGE     VERSION
ip-192-168-16-99.ap-northeast-1.compute.internal    Ready    <none>   5h41m   v1.21.2-13+d2965f0db10712
ip-192-168-63-234.ap-northeast-1.compute.internal   Ready    <none>   5h41m   v1.21.2-13+d2965f0db10712
ip-192-168-72-126.ap-northeast-1.compute.internal   Ready    <none>   5h41m   v1.21.2-13+d2965f0db10712
ip-192-168-87-159.ap-northeast-1.compute.internal   Ready    <none>   16m     v1.21.2-13+d2965f0db10712
```

数分後にもう一度確認するとスケールインが実行されていた
```
$ kubectl get nodes
NAME                                                STATUS   ROLES    AGE     VERSION
ip-192-168-16-99.ap-northeast-1.compute.internal    Ready    <none>   5h45m   v1.21.2-13+d2965f0db10712
ip-192-168-63-234.ap-northeast-1.compute.internal   Ready    <none>   5h45m   v1.21.2-13+d2965f0db10712
ip-192-168-87-159.ap-northeast-1.compute.internal   Ready    <none>   20m     v1.21.2-13+d2965f0db10712
```

ASGの設定を変更してみる
```
aws autoscaling \
    update-auto-scaling-group \
    --auto-scaling-group-name ${ASG_NAME} \
    --min-size 1 \
    --desired-capacity 1 \
    --max-size 3
```
```
 $ aws autoscaling \
>     describe-auto-scaling-groups \
>     --query "AutoScalingGroups[? Tags[? (Key=='eks:cluster-name') && Value=='eksworkshop-eksctl']].[AutoScalingGroupName, MinSize, MaxSize,DesiredCapacity]" \
>     --output table
-----------------------------------------------------------------------------------
|                            DescribeAutoScalingGroups                            |
+------------------------------------------------------------------+----+----+----+
|  eks-eksworkshop-nodegroup-20c1e75b-20f3-9681-9315-f3c70949475e  |  1 |  3 |  1 |
+------------------------------------------------------------------+----+----+----+
```

期待通りnodeが1つに
```
$ kubectl get nodes
NAME                                                STATUS                     ROLES    AGE     VERSION
ip-192-168-16-99.ap-northeast-1.compute.internal    Ready,SchedulingDisabled   <none>   5h59m   v1.21.2-13+d2965f0db10712
ip-192-168-63-234.ap-northeast-1.compute.internal   Ready,SchedulingDisabled   <none>   5h59m   v1.21.2-13+d2965f0db10712
ip-192-168-87-159.ap-northeast-1.compute.internal   Ready                      <none>   34m     v1.21.2-13+d2965f0db10712
```
```
$ kubectl get nodes
NAME                                                STATUS   ROLES    AGE   VERSION
ip-192-168-87-159.ap-northeast-1.compute.internal   Ready    <none>   37m   v1.21.2-13+d2965f0db10712
```

* スケールインの対象外になる条件について
  * [CAのFAQ](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-types-of-pods-can-prevent-ca-from-removing-a-node)に記載あり
    * 対象Nodeに載っているPodがPodDisruptionBudget(以下PDB)によってEvictされることを制限されている
    * kube-system namespace配下のPodが存在している、かつ
      * そのPodはデフォルトで起動しないPodである
      * そのPodにPDBが設定されていない、またはPDBの制限が厳しすぎる
    * DeploymentやStatefulSetなどのController Objectの管理外のPodが存在している
    * localStorageを持つPodが存在している
    * ソース不足などでPodがEvictできないとき
    * 以下のアノテーション付与されているPodが存在している
      * "cluster-autoscaler.kubernetes.io/safe-to-evict": "false"

# クリーンアップ

```
$ kubectl delete -f ~/environment/cluster-autoscaler/nginx.yaml
deployment.apps "nginx-to-scaleout" deleted
```

```
$ kubectl delete -f https://www.eksworkshop.com/beginner/080_scaling/deploy_ca.files/cluster-autoscaler-autodiscover.yaml
clusterrole.rbac.authorization.k8s.io "cluster-autoscaler" deleted
role.rbac.authorization.k8s.io "cluster-autoscaler" deleted
clusterrolebinding.rbac.authorization.k8s.io "cluster-autoscaler" deleted
rolebinding.rbac.authorization.k8s.io "cluster-autoscaler" deleted
deployment.apps "cluster-autoscaler" deleted
```
```
$ eksctl delete iamserviceaccount \
>   --name cluster-autoscaler \
>   --namespace kube-system \
>   --cluster eksworkshop-eksctl \
>   --wait
2022-10-13 07:32:12 [ℹ]  eksctl version 0.61.0
2022-10-13 07:32:12 [ℹ]  using region ap-northeast-1
2022-10-13 07:32:13 [ℹ]  1 iamserviceaccount (kube-system/cluster-autoscaler) was included (based on the include/exclude rules)
2022-10-13 07:32:13 [ℹ]  1 task: { 2 sequential sub-tasks: { delete IAM role for serviceaccount "kube-system/cluster-autoscaler", delete serviceaccount "kube-system/cluster-autoscaler" } }
2022-10-13 07:32:14 [ℹ]  will delete stack "eksctl-eksworkshop-eksctl-addon-iamserviceaccount-kube-system-cluster-autoscaler"
2022-10-13 07:32:14 [ℹ]  waiting for stack "eksctl-eksworkshop-eksctl-addon-iamserviceaccount-kube-system-cluster-autoscaler" to get deleted
2022-10-13 07:32:14 [ℹ]  waiting for CloudFormation stack "eksctl-eksworkshop-eksctl-addon-iamserviceaccount-kube-system-cluster-autoscaler"
2022-10-13 07:32:30 [ℹ]  waiting for CloudFormation stack "eksctl-eksworkshop-eksctl-addon-iamserviceaccount-kube-system-cluster-autoscaler"
2022-10-13 07:32:31 [ℹ]  deleted serviceaccount "kube-system/cluster-autoscaler"
```
```
$ aws iam delete-policy \
  --policy-arn arn:aws:iam::${AWS_ACCOUNT}:policy/k8s-asg-policy
```
```
$ cd ~/environment
$ eksctl delete cluster -f cluster-config.yaml --wait
```