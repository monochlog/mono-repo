# VirtualBox

## ホストネットワークマネージャ
### VirtualBox Host-Only Ethernet Adapter

- アダプター
 ```
 アダプターを手動で設定
 IPv4アドレス：192.168.56.1
 IPv4ネットマスク：255.255.255.0
```
- DHCPサーバ
```
サーバアドレス：192.168.56.1
サーバマスク：255.255.255.0
アドレス下限：192.168.56.101
アドレス上限：192.168.56.254
```


## 各VMでのネットワーク設定
### アダプター１
```
NAT
```

### アダプタ-2
```
ホストオンリーアダプター
VirtualBox host-Only Ethernet Adapter
```

