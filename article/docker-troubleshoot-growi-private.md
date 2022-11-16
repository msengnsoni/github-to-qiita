---
title: "growi障害対応報告"
topics: ["Growi", "トラブルシューティング"]
published: false
---

# はじめに

限定共有記事なので他社の人間には見られないはずですが、
弊社のセキュリティ/コンプライアンス上問題がある場合本記事は削除します。

復旧にあたり紆余曲折あったので、自戒も込めて経緯を残したいと思い書きました。

# 結論

**docker-composeから停止し、再度構築コマンドを流した結果復旧しました。**
ただ、後述しますがこの再構築コマンドは途中で中断しています。

# 報告概要

- [はじめに](#はじめに)
- [結論](#結論)
- [報告概要](#報告概要)
  - [前提:初期調査時に判明したこと](#前提初期調査時に判明したこと)
  - [不要なimageを削除](#不要なimageを削除)
  - [誤ったdocker-compose.ymlを使用し再起動(再構築)](#誤ったdocker-composeymlを使用し再起動再構築)
  - [誤ったymlで再構築されたコンテナを削除](#誤ったymlで再構築されたコンテナを削除)
  - [正しいdocker-compose.ymlを使用し再起動(再構築)](#正しいdocker-composeymlを使用し再起動再構築)
  - [現在のEC2の状態](#現在のec2の状態)
  - [恒久対応として](#恒久対応として)

## 前提:初期調査時に判明したこと

初期調査段階の状況は、
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
f3c71285aae6        growidockercompose_app                "/sbin/tini -e 143 -…"   2 years ago         Up 5 days                  0.0.0.0:3000->3000/tcp   growidockercompose_app_1
1185d325a353        weseek/mongodb-awesome-backup:0.2.0   "/opt/bin/entrypoint…"   2 years ago         Exited (137) 5 weeks ago                            growidockercompose_backup_1
93619fdb0a76        growidockercompose_elasticsearch      "/usr/local/bin/dock…"   2 years ago         Up 8 days                  9200/tcp, 9300/tcp       growidockercompose_elasticsearch_1
12f1544f0780        mongo:3.6                             "docker-entrypoint.s…"   2 years ago         Exited (255) 8 days ago    27017/tcp                growidockercompose_mongo_1
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

以上とmongoDBコンテナ起動時のエラー内容から、新たにコンテナを起動するスペースがないと判断しました。

## 不要なimageを削除

タグがnoneのimageを削除する方針です。(和氣さんと金曜会話した方式)
上記の`docker system df`の結果から、Inactiveなimageが6つあり、2.119GB無駄に食っている様子です。
`docker image prune`を実行し、どのコンテナからも参照されていないimageを削除したものの、
**1つしかimageが削除されず**期待していたほどの容量の確保には繋がりませんでした。

ただし、**mongoDB用コンテナの起動には成功しました。**
この段階ではgrowiは復旧せず、別の方式を検討。

```bash:docker&nbsp;image&nbsp;prune実行後のimage群
# docker images -a
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
<none>                                          <none>              bdacfc332b6c        2 years ago         258MB
growidockercompose_app                          latest              416097e90070        2 years ago         258MB
<none>                                          <none>              2d624342baa6        2 years ago         250MB
<none>                                          <none>              4e6052a0c896        2 years ago         250MB
<none>                                          <none>              87e38f1805a1        2 years ago         250MB
weseek/growi                                    3                   b4fe9588491b        2 years ago         250MB
growidockercompose_elasticsearch                latest              e1c90b9847bb        2 years ago         860MB
<none>                                          <none>              0db543986e2b        2 years ago         847MB
<none>                                          <none>              9366a7bab0e1        2 years ago         842MB
mongo                                           3.6                 99f2b1a58251        2 years ago         432MB
docker.elastic.co/elasticsearch/elasticsearch   6.6.1               c6ffcb0ee97e        3 years ago         842MB
<none>                                          <none>              bf6c809456bb        3 years ago         250MB
growi_app                                       latest              43f6bc980129        3 years ago         250MB
<none>                                          <none>              929356116dda        3 years ago         242MB
<none>                                          <none>              9c90a34779ac        3 years ago         242MB
mongo                                           3.4                 51dbe74a4794        3 years ago         374MB
<none>                                          <none>              cb2455824d3a        3 years ago         242MB
weseek/mongodb-awesome-backup                   0.2.0               21ccb4ab1fe4        3 years ago         215MB
elasticsearch                                   5.3-alpine          c818119f17a4        4 years ago         123MB
```

:::note info
REPOSITORYとTAGが<none>のimageが複数あるが、何かしらのコンテナに紐付いている模様。
コンテナと関連するimageの確認方法がわからず...ご存知の方いたら教えてください。
:::

## 誤ったdocker-compose.ymlを使用し再起動(再構築)

とにかく一旦再起動を、と思いdocker-composeから再起動することにしました。
どうやらgrowiをpullしてきたディレクトリは、`/home/ec2-user/growi`だと判明。
上記に`cd`し、`Dckerfile`、`docker-compose.yml`の**中身を確認せず**、`docker-compose up`を実行してしまいます。

## 誤ったymlで再構築されたコンテナを削除

その結果、旧verのgrowi用コンテナが新規に作成されたため、該当コンテナを削除することに。
この際、明示的に該当コンテナを停止し既存のコンテナが起動していることを確認した後に
`docker system prune`を実行したのですが、
**既存のapp用コンテナ(growidockercompose_app)も何故か一緒に削除されてしまいました。**
>
ちなみにですが、旧ver=mongoDB v3.4、現ver=MongoDB v3.6を使用する環境です。
また、旧verにはgrowiが提供するバックアップシステム(mongodb-awesome-backup)がありません
>

## 正しいdocker-compose.ymlを使用し再起動(再構築)

ここまで実施後、ようやくEBSのバックアップ取得します。AWS上にあるEBSスナップショットの断面はこの時点となります。
幸いapp層が消えただけだったので、現verのdocker-compose.ymlを実行し元通りになりました。
現verのディレクトリは`/home/growi-docker-compose`です。dbコンテナが消えていたら詰みでした。
結論にも書いていますが、このdocker-compose.yml実行は中断してします。以下実行ログ。

```bash:docker-compose&nbsp;up実行結果
Starting growidockercompose_mongo_1
Starting growidockercompose_elasticsearch_1
Starting growidockercompose_app_1
Starting growidockercompose_backup_1
Attaching to growidockercompose_elasticsearch_1, growidockercompose_mongo_1, growidockercompose_backup_1, growidockercompose_app_1
mongo_1          | 2022-02-20T10:12:06.513+0000 I CONTROL  [initandlisten] MongoDB starting : pid=1 port=27017 dbpath=/data/db 64-bit host=12f1544f0780
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten] db version v3.6.13
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten] git version: db3c76679b7a3d9b443a0e1b3e45ed02b88c539f
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten] OpenSSL version: OpenSSL 1.0.2g  1 Mar 2016
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten] allocator: tcmalloc
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten] modules: none
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten] build environment:
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten]     distmod: ubuntu1604
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten]     distarch: x86_64
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten]     target_arch: x86_64
mongo_1          | 2022-02-20T10:12:06.521+0000 I CONTROL  [initandlisten] options: { net: { bindIpAll: true } }
mongo_1          | 2022-02-20T10:12:06.522+0000 I -        [initandlisten] Detected data files in /data/db created by the 'wiredTiger' storage engine, so setting the active storage engine to 'wiredTiger'.
mongo_1          | 2022-02-20T10:12:06.522+0000 I STORAGE  [initandlisten] wiredtiger_open config: create,cache_size=483M,session_max=20000,eviction=(threads_min=4,threads_max=4),config_base=false,statistics=(fast),compatibility=(release="3.0",require_max="3.0"),log=(enabled=true,archive=true,path=journal,compressor=snappy),file_manager=(close_idle_time=100000),statistics_log=(wait=0),verbose=(recovery_progress),
backup_1         | === started in cron mode 2022/02/20 19:12:06 ===
backup_1         | # minute (0-59),
backup_1         | # |  hour (0-23),
backup_1         | # |  |  day of the month (1-31),
backup_1         | # |  |  |  month of the year (1-12),
backup_1         | # |  |  |  |  day of the week (0-6 with 0=Sunday).
backup_1         | # |  |  |  |  |  commands
backup_1         |   0  4  *  *  * /opt/bin/command_exec.sh backup prune list
backup_1         | crond: crond (busybox 1.27.2) started, log level 8
app_1            | 2022/02/20 10:12:06 Waiting for: tcp://mongo:27017
app_1            | 2022/02/20 10:12:06 Waiting for: tcp://elasticsearch:9200
app_1            | 2022/02/20 10:12:06 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:06 Problem with dial: dial tcp 172.18.0.2:27017: connect: connection refused. Sleeping 1s
elasticsearch_1  | OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
app_1            | 2022/02/20 10:12:07 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:07 Problem with dial: dial tcp 172.18.0.2:27017: connect: connection refused. Sleeping 1s
mongo_1          | 2022-02-20T10:12:08.216+0000 I STORAGE  [initandlisten] WiredTiger message [1645351928:215995][1:0x7f612fba3a40], txn-recover: Main recovery loop: starting at 38/768
mongo_1          | 2022-02-20T10:12:08.385+0000 I STORAGE  [initandlisten] WiredTiger message [1645351928:385196][1:0x7f612fba3a40], txn-recover: Recovering log 38 through 39
mongo_1          | 2022-02-20T10:12:08.491+0000 I STORAGE  [initandlisten] WiredTiger message [1645351928:491656][1:0x7f612fba3a40], txn-recover: Recovering log 39 through 39
mongo_1          | 2022-02-20T10:12:08.563+0000 I STORAGE  [initandlisten] WiredTiger message [1645351928:563484][1:0x7f612fba3a40], txn-recover: Set global recovery timestamp: 0
mongo_1          | 2022-02-20T10:12:08.648+0000 I CONTROL  [initandlisten]
mongo_1          | 2022-02-20T10:12:08.649+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
mongo_1          | 2022-02-20T10:12:08.649+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
mongo_1          | 2022-02-20T10:12:08.649+0000 I CONTROL  [initandlisten]
app_1            | 2022/02/20 10:12:08 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:08 Problem with dial: dial tcp 172.18.0.2:27017: connect: connection refused. Sleeping 1s
mongo_1          | 2022-02-20T10:12:08.680+0000 I FTDC     [initandlisten] Initializing full-time diagnostic data capture with directory '/data/db/diagnostic.data'
mongo_1          | 2022-02-20T10:12:08.681+0000 I NETWORK  [initandlisten] waiting for connections on port 27017
app_1            | 2022/02/20 10:12:09 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:09 Connected to tcp://mongo:27017
mongo_1          | 2022-02-20T10:12:09.681+0000 I NETWORK  [listener] connection accepted from 172.18.0.4:48288 #1 (1 connection now open)
app_1            | 2022/02/20 10:12:10 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:11 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
elasticsearch_1  | [2022-02-20T10:12:12,678][INFO ][o.e.e.NodeEnvironment    ] [rlJBfNH] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/xvda1)]], net usable_space [670.9mb], net total_space [9.9gb], types [xfs]
app_1            | 2022/02/20 10:12:12 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
elasticsearch_1  | [2022-02-20T10:12:12,688][INFO ][o.e.e.NodeEnvironment    ] [rlJBfNH] heap size [247.6mb], compressed ordinary object pointers [true]
elasticsearch_1  | [2022-02-20T10:12:12,752][INFO ][o.e.n.Node               ] [rlJBfNH] node name derived from node ID [rlJBfNH6S1CUrNHeqrbj1A]; set [node.name] to override
elasticsearch_1  | [2022-02-20T10:12:12,769][INFO ][o.e.n.Node               ] [rlJBfNH] version[6.6.1], pid[1], build[default/tar/1fd8f69/2019-02-13T17:10:04.160291Z], OS[Linux/4.14.114-103.97.amzn2.x86_64/amd64], JVM[Oracle Corporation/OpenJDK 64-Bit Server VM/11.0.1/11.0.1+13]
elasticsearch_1  | [2022-02-20T10:12:12,770][INFO ][o.e.n.Node               ] [rlJBfNH] JVM arguments [-Xms1g, -Xmx1g, -XX:+UseConcMarkSweepGC, -XX:CMSInitiatingOccupancyFraction=75, -XX:+UseCMSInitiatingOccupancyOnly, -Des.networkaddress.cache.ttl=60, -Des.networkaddress.cache.negative.ttl=10, -XX:+AlwaysPreTouch, -Xss1m, -Djava.awt.headless=true, -Dfile.encoding=UTF-8, -Djna.nosys=true, -XX:-OmitStackTraceInFastThrow, -Dio.netty.noUnsafe=true, -Dio.netty.noKeySetOptimization=true, -Dio.netty.recycler.maxCapacityPerThread=0, -Dlog4j.shutdownHookEnabled=false, -Dlog4j2.disable.jmx=true, -Djava.io.tmpdir=/tmp/elasticsearch-10915902486292266165, -XX:+HeapDumpOnOutOfMemoryError, -XX:HeapDumpPath=data, -XX:ErrorFile=logs/hs_err_pid%p.log, -Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m, -Djava.locale.providers=COMPAT, -XX:UseAVX=2, -Des.cgroups.hierarchy.override=/, -Xms256m, -Xmx256m, -Des.path.home=/usr/share/elasticsearch, -Des.path.conf=/usr/share/elasticsearch/config, -Des.distribution.flavor=default, -Des.distribution.type=tar]
app_1            | 2022/02/20 10:12:13 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:14 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:15 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:16 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
elasticsearch_1  | [2022-02-20T10:12:16,797][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [aggs-matrix-stats]
elasticsearch_1  | [2022-02-20T10:12:16,798][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [analysis-common]
elasticsearch_1  | [2022-02-20T10:12:16,799][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [ingest-common]
elasticsearch_1  | [2022-02-20T10:12:16,799][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [lang-expression]
elasticsearch_1  | [2022-02-20T10:12:16,799][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [lang-mustache]
elasticsearch_1  | [2022-02-20T10:12:16,800][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [lang-painless]
elasticsearch_1  | [2022-02-20T10:12:16,800][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [mapper-extras]
elasticsearch_1  | [2022-02-20T10:12:16,801][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [parent-join]
elasticsearch_1  | [2022-02-20T10:12:16,802][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [percolator]
elasticsearch_1  | [2022-02-20T10:12:16,802][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [rank-eval]
elasticsearch_1  | [2022-02-20T10:12:16,802][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [reindex]
elasticsearch_1  | [2022-02-20T10:12:16,803][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [repository-url]
elasticsearch_1  | [2022-02-20T10:12:16,804][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [transport-netty4]
elasticsearch_1  | [2022-02-20T10:12:16,804][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [tribe]
elasticsearch_1  | [2022-02-20T10:12:16,805][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-ccr]
elasticsearch_1  | [2022-02-20T10:12:16,805][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-core]
elasticsearch_1  | [2022-02-20T10:12:16,805][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-deprecation]
elasticsearch_1  | [2022-02-20T10:12:16,806][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-graph]
elasticsearch_1  | [2022-02-20T10:12:16,807][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-ilm]
elasticsearch_1  | [2022-02-20T10:12:16,807][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-logstash]
elasticsearch_1  | [2022-02-20T10:12:16,808][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-ml]
elasticsearch_1  | [2022-02-20T10:12:16,808][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-monitoring]
elasticsearch_1  | [2022-02-20T10:12:16,809][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-rollup]
elasticsearch_1  | [2022-02-20T10:12:16,809][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-security]
elasticsearch_1  | [2022-02-20T10:12:16,809][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-sql]
elasticsearch_1  | [2022-02-20T10:12:16,811][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-upgrade]
elasticsearch_1  | [2022-02-20T10:12:16,811][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded module [x-pack-watcher]
elasticsearch_1  | [2022-02-20T10:12:16,816][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded plugin [analysis-icu]
elasticsearch_1  | [2022-02-20T10:12:16,817][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded plugin [analysis-kuromoji]
elasticsearch_1  | [2022-02-20T10:12:16,817][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded plugin [ingest-geoip]
elasticsearch_1  | [2022-02-20T10:12:16,820][INFO ][o.e.p.PluginsService     ] [rlJBfNH] loaded plugin [ingest-user-agent]
app_1            | 2022/02/20 10:12:17 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:18 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:19 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:20 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:21 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:22 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:23 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:24 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:25 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
elasticsearch_1  | [2022-02-20T10:12:26,597][INFO ][o.e.x.s.a.s.FileRolesStore] [rlJBfNH] parsed [0] roles from file [/usr/share/elasticsearch/config/roles.yml]
app_1            | 2022/02/20 10:12:26 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
elasticsearch_1  | [2022-02-20T10:12:27,631][INFO ][o.e.x.m.p.l.CppLogMessageHandler] [rlJBfNH] [controller/71] [Main.cc@109] controller (64 bit): Version 6.6.1 (Build a033f1b9679cab) Copyright (c) 2019 Elasticsearch BV
app_1            | 2022/02/20 10:12:27 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:28 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
elasticsearch_1  | [2022-02-20T10:12:29,174][INFO ][o.e.d.DiscoveryModule    ] [rlJBfNH] using discovery type [zen] and host providers [settings]
app_1            | 2022/02/20 10:12:29 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:30 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
elasticsearch_1  | [2022-02-20T10:12:30,861][INFO ][o.e.n.Node               ] [rlJBfNH] initialized
elasticsearch_1  | [2022-02-20T10:12:30,862][INFO ][o.e.n.Node               ] [rlJBfNH] starting ...
elasticsearch_1  | [2022-02-20T10:12:31,163][INFO ][o.e.t.TransportService   ] [rlJBfNH] publish_address {127.0.0.1:9300}, bound_addresses {127.0.0.1:9300}
elasticsearch_1  | [2022-02-20T10:12:31,262][WARN ][o.e.b.BootstrapChecks    ] [rlJBfNH] max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
elasticsearch_1  | [2022-02-20T10:12:31,264][WARN ][o.e.b.BootstrapChecks    ] [rlJBfNH] max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
app_1            | 2022/02/20 10:12:31 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:32 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
app_1            | 2022/02/20 10:12:33 Problem with dial: dial tcp 172.18.0.3:9200: connect: connection refused. Sleeping 1s
elasticsearch_1  | [2022-02-20T10:12:34,372][INFO ][o.e.c.s.MasterService    ] [rlJBfNH] zen-disco-elected-as-master ([0] nodes joined), reason: new_master {rlJBfNH}{rlJBfNH6S1CUrNHeqrbj1A}{4DpwKOv6RWySP8oFhHPxPQ}{127.0.0.1}{127.0.0.1:9300}{ml.machine_memory=2088378368, xpack.installed=true, ml.max_open_jobs=20, ml.enabled=true}
elasticsearch_1  | [2022-02-20T10:12:34,377][INFO ][o.e.c.s.ClusterApplierService] [rlJBfNH] new_master {rlJBfNH}{rlJBfNH6S1CUrNHeqrbj1A}{4DpwKOv6RWySP8oFhHPxPQ}{127.0.0.1}{127.0.0.1:9300}{ml.machine_memory=2088378368, xpack.installed=true, ml.max_open_jobs=20, ml.enabled=true}, reason: apply cluster state (from master [master {rlJBfNH}{rlJBfNH6S1CUrNHeqrbj1A}{4DpwKOv6RWySP8oFhHPxPQ}{127.0.0.1}{127.0.0.1:9300}{ml.machine_memory=2088378368, xpack.installed=true, ml.max_open_jobs=20, ml.enabled=true} committed version [1] source [zen-disco-elected-as-master ([0] nodes joined)]])
elasticsearch_1  | [2022-02-20T10:12:34,481][INFO ][o.e.h.n.Netty4HttpServerTransport] [rlJBfNH] publish_address {172.18.0.3:9200}, bound_addresses {0.0.0.0:9200}
elasticsearch_1  | [2022-02-20T10:12:34,482][INFO ][o.e.n.Node               ] [rlJBfNH] started
app_1            | 2022/02/20 10:12:34 Connected to tcp://elasticsearch:9200
app_1            |
app_1            | > growi@3.6.6 preserver:prod /opt/growi
app_1            | > npm run migrate
app_1            |
app_1            |
app_1            | > growi@3.6.6 migrate /opt/growi
app_1            | > npm run migrate:up
app_1            |
app_1            |
app_1            | > growi@3.6.6 migrate:up /opt/growi
app_1            | > migrate-mongo up -f config/migrate.js
app_1            |
elasticsearch_1  | [2022-02-20T10:12:38,130][WARN ][o.e.x.s.a.s.m.NativeRoleMappingStore] [rlJBfNH] Failed to clear cache for realms [[]]
elasticsearch_1  | [2022-02-20T10:12:38,268][INFO ][o.e.l.LicenseService     ] [rlJBfNH] license [f0b37dc2-dd1a-4910-80dd-ab316052ab7c] mode [basic] - valid
elasticsearch_1  | [2022-02-20T10:12:38,302][INFO ][o.e.g.GatewayService     ] [rlJBfNH] recovered [2] indices into cluster_state
app_1            | (node:52) DeprecationWarning: current Server Discovery and Monitoring engine is deprecated, and will be removed in a future version. To use the new Server Discover and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.
mongo_1          | 2022-02-20T10:12:38.419+0000 I NETWORK  [listener] connection accepted from 172.18.0.4:48364 #2 (2 connections now open)
mongo_1          | 2022-02-20T10:12:38.426+0000 I NETWORK  [conn2] received client metadata from 172.18.0.4:48364 conn2: { driver: { name: "nodejs", version: "3.3.3" }, os: { type: "Linux", name: "linux", architecture: "x64", version: "4.14.114-103.97.amzn2.x86_64" }, platform: "Node.js v12.14.1, LE, mongodb-core: 3.3.3" }
mongo_1          | 2022-02-20T10:12:38.480+0000 I NETWORK  [conn2] end connection 172.18.0.4:48364 (1 connection now open)
app_1            |
app_1            | > growi@3.6.6 server:prod /opt/growi
app_1            | > env-cmd -f config/env.prod.js node src/server/app.js
app_1            |
mongo_1          | 2022-02-20T10:12:39.641+0000 I NETWORK  [listener] connection accepted from 172.18.0.4:48366 #3 (2 connections now open)
mongo_1          | 2022-02-20T10:12:39.653+0000 I NETWORK  [conn3] received client metadata from 172.18.0.4:48366 conn3: { driver: { name: "nodejs", version: "3.1.10" }, os: { type: "Linux", name: "linux", architecture: "x64", version: "4.14.114-103.97.amzn2.x86_64" }, platform: "Node.js v12.14.1, LE, mongodb-core: 3.1.9" }
app_1            | express-validator: requires to express-validator/check are deprecated.You should just use require("express-validator") instead.
app_1            | (node:70) DeprecationWarning: collection.ensureIndex is deprecated. Use createIndexes instead.
mongo_1          | 2022-02-20T10:12:40.157+0000 I NETWORK  [listener] connection accepted from 172.18.0.4:48368 #4 (3 connections now open)
mongo_1          | 2022-02-20T10:12:40.159+0000 I NETWORK  [listener] connection accepted from 172.18.0.4:48370 #5 (4 connections now open)
mongo_1          | 2022-02-20T10:12:40.165+0000 I NETWORK  [conn4] received client metadata from 172.18.0.4:48368 conn4: { driver: { name: "nodejs", version: "3.2.7" }, os: { type: "Linux", name: "linux", architecture: "x64", version: "4.14.114-103.97.amzn2.x86_64" }, platform: "Node.js v12.14.1, LE, mongodb-core: 3.2.7" }
mongo_1          | 2022-02-20T10:12:40.198+0000 I NETWORK  [listener] connection accepted from 172.18.0.4:48372 #6 (5 connections now open)
mongo_1          | 2022-02-20T10:12:40.207+0000 I NETWORK  [listener] connection accepted from 172.18.0.4:48374 #7 (6 connections now open)
mongo_1          | 2022-02-20T10:12:40.215+0000 I NETWORK  [listener] connection accepted from 172.18.0.4:48376 #8 (7 connections now open)
elasticsearch_1  | [2022-02-20T10:12:40,322][INFO ][o.e.c.r.a.AllocationService] [rlJBfNH] Cluster health status changed from [RED] to [YELLOW] (reason: [shards started [[growi][0]] ...]).
app_1            | [2022-02-20T10:12:41.883Z]  INFO: growi:service:search/70 on 54febcc02cea: Initializing search delegator
app_1            | [2022-02-20T10:12:41.885Z]  INFO: growi:service:search/70 on 54febcc02cea: Elasticsearch (not Searchbox) is enabled
app_1            | [2022-02-20T10:12:44.269Z]  INFO: growi:plugins:PluginService/70 on 54febcc02cea: load plugin 'growi-plugin-attachment-refs'
app_1            | [2022-02-20T10:12:44.273Z]  INFO: growi:plugins:PluginService/70 on 54febcc02cea: load plugin 'growi-plugin-lsx'
app_1            | [2022-02-20T10:12:44.278Z]  INFO: growi:plugins:PluginService/70 on 54febcc02cea: load plugin 'growi-plugin-pukiwiki-like-linker'
app_1            | express-validator: requires to express-validator/filter are deprecated.You should just use require("express-validator") instead.
app_1            | [2022-02-20T10:12:44.740Z]  INFO: growi:crowi/70 on 54febcc02cea: [production] Express server is listening on port 3000
elasticsearch_1  | [2022-02-20T10:13:04,496][WARN ][o.e.c.r.a.DiskThresholdMonitor] [rlJBfNH] high disk watermark [90%] exceeded on [rlJBfNH6S1CUrNHeqrbj1A][rlJBfNH][/usr/share/elasticsearch/data/nodes/0] free: 671.6mb[6.5%], shards will be relocated away from this node
elasticsearch_1  | [2022-02-20T10:13:04,499][INFO ][o.e.c.r.a.DiskThresholdMonitor] [rlJBfNH] rerouting shards: [high disk watermark exceeded on one or more nodes]
elasticsearch_1  | [2022-02-20T10:13:34,518][WARN ][o.e.c.r.a.DiskThresholdMonitor] [rlJBfNH] high disk watermark [90%] exceeded on [rlJBfNH6S1CUrNHeqrbj1A][rlJBfNH][/usr/share/elasticsearch/data/nodes/0] free: 671.6mb[6.5%], shards will be relocated away from this node
elasticsearch_1  | [2022-02-20T10:14:04,522][WARN ][o.e.c.r.a.DiskThresholdMonitor] [rlJBfNH] high disk watermark [90%] exceeded on [rlJBfNH6S1CUrNHeqrbj1A][rlJBfNH][/usr/share/elasticsearch/data/nodes/0] free: 671.6mb[6.5%], shards will be relocated away from this node
~~以下、下から3つのログの繰り返し~~
```

※実行ログは報告メールに添付します。
ログを少し調べたところ、Elasticserch用の容量が確保できていない状態のように思われました。
Elasticserchが完全に起動しきれていないはずですが、**ここでなぜかgrowwikiが復活しました。**

## 現在のEC2の状態

- ストレージ状況

```bash
# df -lh
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        979M     0  979M   0% /dev
tmpfs           996M     0  996M   0% /dev/shm
tmpfs           996M  592K  996M   1% /run
tmpfs           996M     0  996M   0% /sys/fs/cgroup
/dev/xvda1       10G  9.3G  752M  93% /
tmpfs           200M     0  200M   0% /run/user/1000
overlay          10G  9.3G  752M  93% /var/lib/docker/overlay2/dfae571094edccb5fb1db7ac08b17a4a1de9e6bc19bcb16e9d3660dd9477b48b/merged
shm              64M     0   64M   0% /var/lib/docker/containers/54febcc02ceab5ce0ff49f11b867c1341293ff87bbcca5157a4447243c7f843c/mounts/shm
overlay          10G  9.3G  752M  93% /var/lib/docker/overlay2/02b4992bc313c1c905ef3ad1477f0d7e346de53fa5e10c6294760618f1bc8015/merged
shm              64M     0   64M   0% /var/lib/docker/containers/93619fdb0a767352068c02a520a9f56bfdee1ffef2dc7a6c6b2648523231e76e/mounts/shm
overlay          10G  9.3G  752M  93% /var/lib/docker/overlay2/0272a2ed848a763ea2394133e7ceead7728aa3f69bad5a0e279e79b00c44cb46/merged
shm              64M     0   64M   0% /var/lib/docker/containers/12f1544f0780cb3d5f1d0c9488592805d0f0534b0f5ec1acfc3585e5e749db37/mounts/shm
```

docker-composeを複数回実行した結果、複数のファイルシステムが作成されています。
今後容量が増えていくファイルシステムが使用されているものだと思うので判明次第、
その他のファイルシステムは削除しようと思います。

- Dockerの状況

```bash:docker&nbsp;ps
# docker ps -a
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS                     PORTS                    NAMES
54febcc02cea        growidockercompose_app                "/sbin/tini -e 143 -…"   3 hours ago         Up 2 hours                 0.0.0.0:3000->3000/tcp   growidockercompose_app_1
1185d325a353        weseek/mongodb-awesome-backup:0.2.0   "/opt/bin/entrypoint…"   2 years ago         Exited (137) 2 hours ago                            growidockercompose_backup_1
93619fdb0a76        growidockercompose_elasticsearch      "/usr/local/bin/dock…"   2 years ago         Up 2 hours                 9200/tcp, 9300/tcp       growidockercompose_elasticsearch_1
12f1544f0780        mongo:3.6                             "docker-entrypoint.s…"   2 years ago         Up 2 hours                 27017/tcp                growidockercompose_mongo_1
```

```bash:imageの状況

# docker images -a

growidockercompose_app                          latest              416097e90070        2 years ago         258MB
<none>                                          <none>              bdacfc332b6c        2 years ago         258MB
<none>                                          <none>              2d624342baa6        2 years ago         250MB
<none>                                          <none>              4e6052a0c896        2 years ago         250MB
<none>                                          <none>              87e38f1805a1        2 years ago         250MB
weseek/growi                                    3                   b4fe9588491b        2 years ago         250MB
growidockercompose_elasticsearch                latest              e1c90b9847bb        2 years ago         860MB
<none>                                          <none>              0db543986e2b        2 years ago         847MB
<none>                                          <none>              9366a7bab0e1        2 years ago         842MB
mongo                                           3.6                 99f2b1a58251        2 years ago         432MB
docker.elastic.co/elasticsearch/elasticsearch   6.6.1               c6ffcb0ee97e        3 years ago         842MB
growi_app                                       latest              43f6bc980129        3 years ago         250MB
<none>                                          <none>              bf6c809456bb        3 years ago         250MB
<none>                                          <none>              929356116dda        3 years ago         242MB
<none>                                          <none>              9c90a34779ac        3 years ago         242MB
mongo                                           3.4                 51dbe74a4794        3 years ago         374MB
<none>                                          <none>              cb2455824d3a        3 years ago         242MB
weseek/mongodb-awesome-backup                   0.2.0               21ccb4ab1fe4        3 years ago         215MB
elasticsearch                                   5.3-alpine          c818119f17a4        4 years ago         123MB
```

```bash:docker&nbsp;system&nbsp;df結果

# docker system df

Images              9                   4                   2.512GB             1.84GB (73%)
Containers          4                   3                   2.416GB             525.6MB (21%)
Local Volumes       5                   0                   331.3MB             331.3MB (100%)
Build Cache         0                   0                   0B                  0B
```

再構築によりapp用コンテナ(growidockercompose_app)のみ新規構築されてます。
それ以外は既存のものが利用された様子。

## 恒久対応として

今とりあえず復旧はしていますが、EBSの容量拡張(ひとまず10GB→15GBにして様子見)を提案します。

正確な原因を特定できなかったのでなんとも言えないのですが、
容量を食っていると思われるDokcerのimageを削除する方針はうまく機能しませんでした。
`docker images -a`で表示されるnoneのimageは、おそらく`docker image prune -a`を使うと
削除されると思いますが、何かしらのコンテナから参照されている以上、どのような影響が出るか不明です。
ここに関しては、私のDockerに対する知識不足です。すみません。
以下公式の-aオプションの説明に記載の、参照なしと未使用の違いがわからず...

<https://matsuand.github.io/docs.docker.jp.onthefly/engine/reference/commandline/system_prune/>
