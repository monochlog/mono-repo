# pacemaker インストールメモ
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
# pcs cluster start -all
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




- 管理webページへのアクセス（GUI）確認のみ
https://@@.@@.@@.@@:2224

pgcluster > 上側メニューでRESOURCE > +Add

①pgsql
```
Resource ID pgsql-svc
 Class/Provider ocf:heartbeat
 Type   pgsql
  -- Optional Arguments
 pgctl  /usr/pgsql-13/bin/pg_ctl
 psql  /usr/pgsql-13/bin/psql
 pgdata /var/lib/pgsql/13/data
 pgdba  postgres
 pgport 5432
 pglibs /usr/pgsql-13/lib
 config /var/lib/pgsql/13/data/postgresql.conf
 pgdb   sample
 logfile    /var/log/pgsql
 repuser    rep_user
```
②gw-ping
```
Resource ID gw-ping
 Class/Provider ocf:pacemaker
 Type   ping
  -- Required Arguments
 host_list  192.168.56.1
```
③dbsvr-vip
```
Resource ID gw-ping
 Class/Provider ocf:hearbeat
 Type   IPaddr2
  -- Required Arguments
 ip  192.168.56.101
  -- Optional Arguments
 nic enp0s4
```

### フェンス機能(STONITH)の設定






## プライマリ設定
`cp /etc/corosync/corosync.conf.example /etc/corosync/corosync.conf`

`vi /etc/corosync/corosync.conf`

`corosync-keygen -l`

`scp -p /etc/corosync/authkey root@192.168.56.13:/etc/corosync/authkey`

`scp -p /etc/corosync/corosync.conf root@192.168.56.13:/etc/corosync/corosync.conf`
```