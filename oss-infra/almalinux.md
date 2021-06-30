# alma linux パラメータ設定(Web#1)

## hostname設定
>hostnamectl set-hostname @@@@<br>
>@@@@はサーバ毎。

## selinux無効化
/etc/selinux/config
>SELINUX=disabled

## firewall無効化
>systemctl stop firewalld<br>
>systemctl disabled firewalld

## ネットワーク設定

/etc/sysconfig/network-scripts/ifcig-enp0s3

>IPADDR=xx.xx.xx.xx<br>
>PREFIX=24<br>
>必要に応じてipv6をnoにする。


