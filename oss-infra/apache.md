# apache パラメータシート

## install
`dnf isntall httpd`
`dnf isntall mod-ssl` #SSL対応

## 設定

### /etc/httpd/conf/httpd.conf
```
ServerAdmin ttt@mmmm.com   #必要であれば管理者のメールアドレスに変更
ServerName websvr10.local:80   #サーバ名指定
Options FollowSymLinks   #indexesの無効化
AllowOverride noe  #.htaccess は利用しない
ServerTokens Prod  #サーバーの応答ヘッダ
```
``` #https用追記
<VirtualHost *:80>
  ServerName example.com:80
  RewriteEngine on
  RewriteCond %{HTTP_HOST} ^example\.com
  RewriteRule ^/(.*)$ https://example.com/$1 [R=301,L]
</VirtualHost>
```

