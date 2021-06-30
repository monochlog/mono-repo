# alma linux パラメータ設定(Web#1)

## keylayout設定
`localectl set-keymap jp106` #日本語キーボードの有効化

## ロケール設定
 `dnf install glibc-langpack-ja` #日本語パックの導入<br>
 `localectl set-locale LANG=ja_JP.UTF8`

## hostname設定

`hostnamectl set-hostname @@@@` #@@@@はサーバ毎。

## selinux無効化
- /etc/selinux/config
```
SELINUX=disabled
```

## firewall無効化
```
systemctl stop firewalld
systemctl disabled firewalld
```

## ネットワーク設定
- /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

```

## sshd設定
```
port 22 #コメントアウトを外す
```

