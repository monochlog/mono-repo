# alma linux パラメータ設定(Web#1)

## hostname設定
>hostnamectl set-hostname @@@@<br>
>@@@@はサーバ毎。

## selinux無効化
/etc/selinux/config
>SELINUX=disabled

## ネットワーク設定

/etc/sysconfig/network-scripts/ifcig-enp0s3

>IPADDR=xx.xx.xx.@@<br>
>PREFIX=24



