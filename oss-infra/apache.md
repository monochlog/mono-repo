# apache 構築メモ

## install
`dnf install httpd`

`dnf install mod-ssl` #SSL対応

## 設定

### /etc/httpd/conf/httpd.conf

リポジトリ conf/httpd.confに記載
## /etc/httpd/conf.d/ssl.conf
リポジトリ /conf/ssl.confに記載
## サーバ証明書の作成(自己証明書 san対応)
`mkdir /etc/httpd/svrcrt`  #どこでもいい
`openssl genrsa -aes128 2048 > server.pem`#秘密鍵の作成、任意のパスフレーズを入力

`openssl req -new -key server.pem > server.csr`#公開鍵の作成
```
Enter pass phrase ofr server.key:（任意）
You are about to be asked to enter information that will be incorporated 
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
fi you enter '.', the field will be left blank.
Country Name (2 letter code) [AU]: JP
State or Province Name (full name) [Some-state]: Tokyo
Locality Name (eg, city) []: Shibadaimon, Minato-ku
Organization Name (eg, company) [Internet Widgits Pty Ltd]: Test Inc.
Organizational Unit Name (eg, section) []:test section
Common Name (eg, YOUR name) []:xx.xx.xx.xx(もしくはURL)
Email Address []:（任意）


Pleaseenter thee following 'extra' attributes to be sent with your certificate request
A challenge password[]:（任意）
An optional company name []:（任意）
```

事前にsan用のファイルを作成する

`vi /etc/host/svrcrt/san`
```
subjectAltName = IP:192.168.56.10,DNS:*.local
```

`openssl x509 -req -days 365 -sha256 -in server.csr -signkey server.pem -out server.crt -extfile san` # サーバ証明書の発行(san含む)

`mv server.pem server.pem.back`# パスフレーズの省略化前バックアップ<br>
`openssl rsa -in server.pem.back -out server.pem` # パスフレーズの省略化