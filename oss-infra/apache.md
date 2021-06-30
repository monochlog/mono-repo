# apache パラメータシート

## install
>#dnf isntall httpd

## 設定

### /etc/httpd/conf/httpd.conf
>ServerAdmin ttt@mmmm.com   #管理者アドレスの変更<br>
>ServerName websvr10.local:80   #サーバ名指定<br>
>Options FollowSymLinks   #indexesの無効化<br>
>AllowOverride All  #.htaccess で設定可能なものは全て有効<br>
>ServerTokens Prod  #サーバーの応答ヘッダ<br>