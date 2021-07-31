# pacemaker インストールメモ -- まともに使えないので利用しません。

導入先はDBサーバ(マスター:00、スレーブ:01)

pacemaker2.0 + corosyn + fence-agents(stonith)で実装

### インストール(共通)
Almalinux 8.3ではデフォルトでリポジトリが無効になっているため、有効化する。

確認コマンド

```shell
# dnf repolist all
```
-- ha(HighAvailability) が無効化されている

- 確認コマンド 

```shell
# dnf info --enablerepo=ha pacemaker fence-agents-all pcs
```

- インストール

```shell
# dnf install --enablerepo=ha pacemaker fence-agents-all pcs
```

- クラスター管理ユーザのパスワード設定
```shell
# passwd hacluster
```

- PCSサービスの起動

```shell
# systemctl start pcsd
```



---
### クラスター設定(Primary svrのみで実施)

- ホストの追加

```shell
# pcs host auth Dbsvr00 addr=192.168.56.12 -u hacluster
# pcs host auth Dbsvr01 addr=192.168.56.13 -u hacluster

```
- クラスターの開始
```shell
# pcs cluster setup pgcluster Dbsvr00 Dbsvr01 --start --enable --force
```
- 確認

```shell
# pcs status
```

- クラスタの開始
```shell
# pcs cluster start
```
- クラスターオプション設定
いったんstonithは利用しない
```shell
# pcs property set stonith-enabled=false
```
2台構成の場合quorumは不要
```shell
# pcs property set no-quorum-policy=ignore
```

```shell
# pcs resource defaults update resource-stickiness="INFINITY"
# pcs resource defaults update migration-threshold="1"
```
---

### リソースの設定
- vipの作成
```shell
# pcs resource create dbsvr-vip ocf:heartbeat:IPaddr2 ip=192.168.56.101 cidr_netmask=24 op monitor interval=30s
```

- postgresqlのリソース作成
```shell 
# pcs resource create pgsql-svc ocf:heartbeat:pgsql pgctl="/usr/pgsql-13/bin/pg_ctl" psql="/usr/pgsql-13/bin/psql" pgdata="/var/lib/pgsql/13/data" pgdba="postgres" pgport="5432" pglibs="/usr/pgsql-13/lib" config="/var/lib/pgsql/13/data/postgresql.conf" logfile="/dev/null" rep_mode="sync" node_list="Dbsvr00 DbSvr01" master_ip=192.168.56.101 repuser=rep_user restart_on_promote=true primary_conninfo_opt="keepalives_idle=60 keepalives_interval=5 keepalives_count=5"
```


```shell
# pcs resource promotable pgsql-svc master-max=1 master-node-max=1 clone-max=2 clone-node-max=1 notify=true
# pcs constraint colocation add dbsvr-vip with Master pgsql-svc-clone INFINITY
# pcs constraint order promote pgsql-svc-clone then start dbsvr-vip symmetrical=false score=INFINITY
# pcs constraint order demote pgsql-svc-clone then stop dbsvr-vip symmetrical=false score=0
```  

- 結果確認
```shell
# pcs status --full
```

- 管理webページへのアクセス（GUI）確認のみ
https://@@.@@.@@.@@:2224

### フェンス機能(STONITH)の設定
