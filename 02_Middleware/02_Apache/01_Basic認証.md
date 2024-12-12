## 1. 設定手順

### 1-1. 対象ディレクトリの AllowOverride が All or AuthConfig になっていることを確認
```
# grep "AllowOverride" /etc/httpd/conf.d/01_example.com.conf
AllowOverride All
```

### 1-2. Apacheの設定ファイルでも指定可能
```
# vi /var/www/vhosts/example.com/.htaccess
-----------------------------------------------------------------------------
AuthUserFile  /etc/httpd/conf.d/.htpasswd
AuthGroupFile /dev/null
AuthName      "Please enter your ID and password"
AuthType      Basic
Require       valid-user
------------------------------------------------------------------------------
# cat /var/www/vhosts/example.com/.htaccess
```

### 1-3. htpasswd調整
```
# htpasswd -c /etc/httpd/conf.d/.htpasswd admin
パスワード入力
```
