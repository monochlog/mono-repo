# apache パラメータシート

## install
`dnf isntall httpd`

## 設定

### /etc/httpd/conf/httpd.conf
```
ServerAdmin ttt@mmmm.com   #管理者アドレスの変更
ServerName websvr10.local:80   #サーバ名指定
Options FollowSymLinks   #indexesの無効化
AllowOverride All  #.htaccess で設定可能なものは全て有効
ServerTokens Prod  #サーバーの応答ヘッダ
```