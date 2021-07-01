# DNSサーバ(BIND)

## インストール
`dnf install bind bind-utils`

## 設定
- /etc/named.conf
```
#追加
acl internal-network {
       10.0.0.0/24;
};

options {
listen-on port 53 {any;};
listen-on-v6 { none；};    #ipv6は使用しない
・・・・
allow-query     { localhost; internal-network;};
・・・・

#最下部に追加

```
