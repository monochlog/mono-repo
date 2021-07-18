# postgresql 構築メモ

## install
`dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm`  #最新rpmを含むリポジトリのインストール      

`dnf -qy module disable postgresql` #osのpostgresqlモジュールの無効化
`dnf install -y postgresql13-server` #インストールしたリポジトリからpostgresqlをインストール

## ユーザの環境変数設定
`su - postgres`

`vi /var/lib/pgsql/.bash_profile`
```追記
PATH=/usr/pgsql-13/bin:$PATH
```

## 初期構築
`su - postgres`
`initdb --encoding=UTF8 --no-locale`
`exit` #postgreユーザを抜ける

### ログディレクトリの作成
`mkdir -p /var/log/pgsql`
`chown postgres:postgres /var/log/pgsql`

### /var/lib/pgsql/13/data/postgres.confの編集
- ログディレクトリ変更
```
log_directory = '/var/log/pgsql/'
```
- 外部接続
```
listen_addresses = '0.0.0.0' # any
port=5432 # default 
```

### /var/lib/pgsql/13/data/pg_hba.confの編集
- sampleDB接続ユーザ(追記)
``` 
host    sampe   all 192.168.56.0/24 md5
```


### 起動(rootユーザ)
`systemctl start postgresql-13`

### 起動確認(postgresユーザ)
`su - postgres`
`psql -l` #DB一覧の表示
`\q` #接続終了

### postgresユーザのパスワード変更
psql -c "ALTER ROLE postgres PASSWORD '1234'"

### データベースの作成(デフォルト設定)
`createdb sample`


### 別サーバからの接続確認
psql -U postgres -h 192.168.56.12 -p 5432 sample


## ストリーミングレプリケーション設定
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

- レプリケーション用ユーザの作成

`su - postgres`<br>
`createuser --replication -P rep_user`

- 対応後にサービス再起動(rootユーザ)

`systemctl restart postgresql-13`

### スレーブでの設定

`systemctl stop postgresql-13` # ルートユーザ。起動している場合。

`rm -rf ${PGDATA}/*`

`pg_basebackup -R -D ${PGDATA} -h 192.168.56.12`

- /var/lib/pgsql/13/data/postgresql.auto.confの修正

`vi /var/lib/pgsql/13/data/postgresql.auto.conf`
```
primary_conninfo = '****(追記) application_name=dbsvr01'
```
- /var/lib/pgsql/13/data/postgresql.confの編集

`vi /var/lib/pgsql/13/data/postgresql.conf`
各種コメントアウト
```
#listen_addresses='0.0.0.0'
#max_wal_senders = 10
#synchronous_standby_names = 'dbsvr01' 
```
