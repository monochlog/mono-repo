# postgresql 構築メモ

## install
```shell
最新rpmを含むリポジトリのインストール      
# dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
osのpostgresqlモジュールの無効化
# dnf -qy module disable postgresql
インストールしたリポジトリからpostgresqlをインストール
# dnf install -y postgresql13-server 
```
## ユーザの環境変数設定
```shell
# su - postgres
#vi /var/lib/pgsql/.bash_profile`
```
```
PATH=/usr/pgsql-13/bin:$PATH
```

## 初期構築
```shell
# su - postgres
# initdb --encoding=UTF8 --no-locale
# exit #postgresユーザを抜ける
```
### ログディレクトリの作成
```shell
# mkdir -p /var/log/pgsql
# chown postgres:postgres /var/log/pgsql
```
### /var/lib/pgsql/13/data/postgres.confの編集
```
log_directory = '/var/log/pgsql/'
listen_addresses = '0.0.0.0' # any
port=5432 # default 
```

### /var/lib/pgsql/13/data/pg_hba.confの編集
- DB接続情報(追記)
``` 
host    all   all 192.168.56.0/24 trust
```


### 起動(rootユーザ)
```shell
# systemctl start postgresql-13
```
### 起動確認(postgresユーザ)
```shell
# su - postgres`
DB一覧の表示
# psql -l
接続終了
# \q 
```
### postgresユーザのパスワード変更
```shell 
# psql -c "ALTER ROLE postgres PASSWORD '1234'"
```
### データベースの作成(デフォルト設定)
```shell
# createdb sample
```


### 別サーバからの接続確認
```shell
# psql -U postgres -h 192.168.56.12 -p 5432 sample
```

---

## ストリーミングレプリケーション設定(利用しないがメモ程度)
マスタ：192.168.56.12
スレーブ：192.168.56.13とする。

### マスタでの設定

- /var/lib/pgsql/13/data/postgresql.confの編集
```
wal_level = replica
synchronous_commit = on
max_wal_senders = 10
synchronous_standby_names = 'dbsvr01' 
#dvsvr01はスレーブ側で指定するapplication_name
```

- /var/lib/pgsql/13/data/pg_hba.confの編集（既存のreplicationを編集）
``` 
host    replication postgres    192.168.56.13/32   trust
```

- 対応後にサービス再起動(rootユーザ)

```shell
# systemctl restart postgresql-13
```
### スレーブでの設定

```shell
# ルートユーザ。起動している場合。
# systemctl stop postgresql-13
# su - postgres
# rm -rf ${PGDATA}/*
# pg_basebackup -D ${PGDATA} -h 192.168.56.12
```

- /var/lib/pgsql/13/data/postgresql.auto.confの修正

```shell
vi /var/lib/pgsql/13/data/postgresql.auto.conf
```

```
primary_conninfo = '****(追記) application_name=dbsvr01'
```
- /var/lib/pgsql/13/data/postgresql.confの編集

```shell
# vi /var/lib/pgsql/13/data/postgresql.conf
```
各種コメントアウト
```
#listen_addresses='0.0.0.0'
#max_wal_senders = 10
#synchronous_standby_names = 'dbsvr01' 
```
- 起動
```shell
# systemctl start postgresql-13
```

- 確認
```shell
# su - postgres
# psql
# select * from pg_stat_replication;
```