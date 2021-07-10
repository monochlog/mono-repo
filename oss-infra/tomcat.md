# tomcat 構築メモ

## install
`dnf install java-11-openjdk-devel.x86_64` #jdk11

`dnf install wget` # wget

`dnf install tar` # tar

## Create tomcat user (非root)

`sudo groupadd tomcat` #ユーザグループ追加

`sudo mkdir /opt/tomcat` 

`sudo useradd -s /bin/nologin -g tomcat -d /opt/tomcat tomcat` # ユーザ追加

`cat /etc/password | grep tomcat ` #ユーザ追加の確認

`wget https://downloads.apache.org/tomcat/tomcat-10/v10.0.8/bin/apache-tomcat-10.0.8.tar.gz` #インストールファイルの取得

`sudo tar -zxvf apache-tomcat-*.tar.gz -C /opt/tomcat --strip-components=1` #展開

`sudo chown -R tomcat: /opt/tomcat` # 権限設定 

`sudo sh -c 'chmod +x /opt/tomcat/bin/*.sh'` 

`sudo vi /etc/systemd/system/tomcat.service` #systemdのユニットファイルの作成
```
[Unit]
Description=Apache Tomcat 10
After=network.target

[Service]
User=tomcat
Group=tomcat
Type=oneshot
PIDFile=/opt/tomcat/tomcat.pid
RemainAfterExit=true

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
ExecReStart=/opt/tomcat/bin/shutdown.sh;/opt/tomcat/bin/startup.sh

[Install]
WantedBy=multi-user.target
```