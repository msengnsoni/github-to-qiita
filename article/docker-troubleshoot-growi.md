---
title: "【docker】EC2の/var/lib/docker/overlay2配下がいっぱいでコンテナが起動できなくなった際の障害対応備忘録"
topics: ["Docker", "Growi", "docker-compose", "トラブルシューティング"]
published: true
---

# はじめに

本記事は、自社内で使用しているgrowiのwikiが起動しなくなったため、
実施した障害対応の経緯を記載したものです。

# 調査時のインスタンスの状態

本サーバ構築者 ≠ 筆者で設計書もないため現状調査から。

**0. OSバージョン**

```bash
$ cat /etc/system-release
Amazon Linux release 2 (Karoo)
```

**1. Dockerのファイルシステムによりボリュームが100%使用状態**

```bash
# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        979M     0  979M   0% /dev
tmpfs           996M     0  996M   0% /dev/shm
tmpfs           996M   65M  932M   7% /run
tmpfs           996M     0  996M   0% /sys/fs/cgroup
/dev/xvda1       10G   10G   40M 100% /
overlay          10G   10G   40M 100% /var/lib/docker/overlay2/02b4992bc313c1c905ef3ad1477f0d7e346de53fa5e10c6294760618f1bc8015/merged
shm              64M     0   64M   0% /var/lib/docker/containers/93619fdb0a767352068c02a520a9f56bfdee1ffef2dc7a6c6b2648523231e76e/mounts/shm
tmpfs           200M     0  200M   0% /run/user/1000
```

**2. mongoDBコンテナが停止状態**

```bash
# docker container ls -a
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS                     PORTS                    NAMES
************        growidockercompose_app                "/sbin/tini -e 143 -…"   2 years ago         Up 5 days                  0.0.0.0:3000->3000/tcp   growidockercompose_app_1
************        weseek/mongodb-awesome-backup:0.2.0   "/opt/bin/entrypoint…"   2 years ago         Exited (137) 5 weeks ago                            growidockercompose_backup_1
************        growidockercompose_elasticsearch      "/usr/local/bin/dock…"   2 years ago         Up 8 days                  9200/tcp, 9300/tcp       growidockercompose_elasticsearch_1
************        mongo:3.6                             "docker-entrypoint.s…"   2 years ago         Exited (255) 8 days ago    27017/tcp                growidockercompose_mongo_1
```

**3. 不要なimageが複数ある(ように見える)**

```bash
# docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              10                  4                   2.791GB             2.119GB (75%)
Containers          4                   2                   2.651GB             525.6MB (19%)
Local Volumes       5                   0                   331.2MB             331.2MB (100%)
Build Cache         0                   0                   0B                  0B
```

以上とmongoDBコンテナ起動時のエラー内容から、新たにコンテナを起動するスペースがないため起動できず、
growiが停止していると判断しました。

# 不要なimageを削除し容量確保

上記の結果から、Inactiveなimageが6つあり、2.119GB無駄に食っている様子です。
`docker image prune`を実行したものの、**1つしかimageが削除されず**
期待していたほどの容量の確保には繋がりませんでしが、
mongoDB用コンテナの起動には成功しました。

ただし、これだけではgrowiが復旧せず、別の方式を検討。

```bash
docker image prune
```

```bash
# docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              9                   4                   2.512GB             1.84GB (73%)
Containers          4                   3                   2.416GB             525.6MB (21%)
Local Volumes       5                   0                   331.3MB             331.3MB (100%)
Build Cache         0                   0                   0B                  0B
```

# 【やらかし】旧verのdocker-compose.ymlを使用し再構築

一旦docker環境の再起動を、と思いdocker-composeから再起動することにしました。
どうやらgrowiをcloneしたディレクトリは、`/home/ec2-user/growi※`だと判明。(やらかし1)

>※後の判明しましたが、上記は旧verのディレクトリでした。
>
>* 旧ver = /home/ec2-user/growi
>* 現ver = /home/growi-docker-compose

ここで私は、`Dockerfile`、`docker-compose.yml`の**中身を確認せず**、
`docker-compose up`を実行します。(やらかし2)

>ちなみにですが、旧ver=mongoDB v3.4、現ver=MongoDB v3.6を使用する環境です。
>また、旧verにはgrowiが提供するバックアップシステム(mongodb-awesome-backup)がありません

# 【やらかし】現verのapp用コンテナが削除

その結果、旧verのgrowi用コンテナが新規に作成されたため、該当コンテナを削除することに。
この際、明示的に該当コンテナを停止し既存のコンテナが起動していることを確認した後に
`docker system prune`を実行したのですが、
**現verのapp用コンテナ(growidockercompose_app)も一緒に削除されてしまいました。**(やらかし3)

```bash
docker system prune
```

```bash
# docker container ls -a
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS                     PORTS                    NAMES
************        weseek/mongodb-awesome-backup:0.2.0   "/opt/bin/entrypoint…"   2 years ago         Exited (137) 5 weeks ago                            growidockercompose_backup_1
************        growidockercompose_elasticsearch      "/usr/local/bin/dock…"   2 years ago         Up 8 days                  9200/tcp, 9300/tcp       growidockercompose_elasticsearch_1
************        mongo:3.6                             "docker-entrypoint.s…"   2 years ago         Exited (255) 8 days ago    27017/tcp                growidockercompose_mongo_1
```

# 現verのdocker-compose.ymlを使用し再構築

幸いapp層のコンテナが消えただけだったので、現verのディレクトリに移動し
`docker-compose up`を実行したら元通りになり、**growiも復旧しました。**
dbコンテナが消えていたら詰んでました。

```bash
docker-compose up
```

# 教訓

* `docker-compose up`は`Dockerfile`、`docker-compose.yml`の中身を確認してから実行しよう！
* `docker system prune`は、開発環境等で動作を確認してから実行しよう！

本記事がどなたかのお役に立てれば幸いです。
