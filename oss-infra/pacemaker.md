# pacemaker インストールメモ
導入先はDBサーバ(マスター:00、スレーブ:01)

pacemaker2.0 + corosynで実装

## インストール(共通)
Almalinux 8.3ではデフォルトでリポジトリが無効になっているため、有効化する。

確認コマンド

```shell
# dnf repolist all
```
-- ha(HighAvailability) が無効化されている

有効化コマンド

```shell
# dnf info --enablerepo=ha pacemaker corosyn pcs
```

確認コマンド

```shell
# dnf info | grep pacemaker
```

```shell
# dnf info | grep corosyn
```

インストール

```shell
# dnf install --enablerepo=ha pacemaker corosync pcs
```

クラスター管理ユーザのパスワード設定
```shell
# passwd hacluster
```

PCSサービスの起動

```shell
# systemctl start pcsd
```

管理webページへのアクセス（GUI）
`https://@@.@@.@@.@@:2224`

ホストの追加

```shell
# pcs host auth 192.168.56.12 192.168.56.13 -u hacluster
```

```
## プライマリ設定
`cp /etc/corosync/corosync.conf.example /etc/corosync/corosync.conf`

`vi /etc/corosync/corosync.conf`

`corosync-keygen -l`

`scp -p /etc/corosync/authkey root@192.168.56.13:/etc/corosync/authkey`

`scp -p /etc/corosync/corosync.conf root@192.168.56.13:/etc/corosync/corosync.conf`
```