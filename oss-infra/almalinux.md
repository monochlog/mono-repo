# alma linux パラメータ設定(Web#1)

### keylayout設定
```shell
# localectl set-keymap jp106
``` 
### ロケール設定 
 ```shell
 # dnf install glibc-langpack-ja
 # localectl set-locale LANG=ja_JP.UTF8
```

### hostname設定

```shell
# hostnamectl set-hostname @@@@` #@@@@はサーバ毎。
```

### selinux無効化
- /etc/selinux/config
```
SELINUX=disabled
```

### firewalld無効化
```shell
# systemctl stop firewalld
```
and
```shell
# systemctl disabled firewalld
```

### ネットワーク設定
- /etc/sysconfig/network-scripts/ifcfg-enp0s8
```shell
# nmtui
```
```
address 192.168.56.xx
gateway 195.168.56.1
DNS servers 195.168.56.1
```

### sshd設定
- /etc/ssh/sshd_config
```
port 22 #コメントアウトを外す
```

