## 1. 異なるPHP Verの同居
### 1-1. 前提条件
#### 1-1-1. OS
```
# cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
```

#### 1-1-2. Apache
```
# httpd -v
Server version: Apache/2.4.6 (CentOS)
Server built:   May 30 2023 14:01:11

# httpd -V | grep "MPM"
Server MPM:     prefork
```

### 1-2. PHPインストール(1バージョン目)
#### 1-2-1. インストール
```
# yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm

# yum repolist all | grep -i "remi"
-----------------------------------------------------------------------------
remi-php73                          Remi's PHP 7.3 RPM repositor disabled
-----------------------------------------------------------------------------
# yum install --enablerepo=remi-php73 php-{cli,common,devel,fedora-autoloader,gd,gmp,json,mbstring,mysqlnd,opcache,pear,pdo,pecl-mcrypt,pecl-zip,process,xml}
# yum install --enablerepo=remi-php73 mod_php


# php -v
PHP 7.3.33 (cli) (built: Jun  5 2024 05:42:20) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.33, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.3.33, Copyright (c) 1999-2018, by Zend Technologies
# httpd -M | grep "php"
 php7_module (shared)

# httpd -t
# systemctl restart httpd
```

#### 1-2-2. 動作テスト①
```
# mkdir /var/www/vhosts/example.net/php73
# echo -e "<?php\nphpinfo();\n?>" > /var/www/vhosts/example.net/php73/phpinfo.php
```

・ブラウザより、実行しているPHPが「PHP Version 7.3」であることを確認
https://example.net/php73/phpinfo.php



### 1-3. PHPインストール(2バージョン目)
#### 1-3-1. インストール
```
# yum install --enablerepo=epel,remi-php80 php80 php80-php-{fpm,cli,common,devel,gd,gmp,json,mbstring,mysqlnd,opcache,pear,pdo,pecl-mcrypt,pecl-zip,process,xml}
# /opt/remi/php80/root/usr/bin/php -v
PHP 8.0.30 (cli) (built: Jun  4 2024 15:13:07) ( NTS gcc x86_64 )
Copyright (c) The PHP Group
Zend Engine v4.0.30, Copyright (c) Zend Technologies
    with Zend OPcache v8.0.30, Copyright (c), by Zend Technologies

# php -v
PHP 7.3.33 (cli) (built: Jun  5 2024 05:42:20) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.33, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.3.33, Copyright (c) 1999-2018, by Zend Technologies
```

#### 1-3-2. php.ini初期設定
```
# cp -ip /etc/opt/remi/php80/php.ini{,.$(date +%Y%m%d)}
# vi /etc/opt/remi/php80/php.ini
下記の通り調整
-----------------------------------------------------------------------------
expose_php = On
;include_path = ".:/php/includes"
;date.timezone =
-----------------------------------------------------------------------------
↓
-----------------------------------------------------------------------------
expose_php = Off
include_path = ".:/opt/remi/php80/root/usr/share/pear:/opt/remi/php80/root/usr/share/php"
date.timezone = "Asia/Tokyo"
-----------------------------------------------------------------------------
# diff /etc/opt/remi/php80/php.ini{,.$(date +%Y%m%d)}
```

#### 1-3-3. PHP-FPM初期設定
```
# cp -ip /etc/opt/remi/php80/php-fpm.d/www.conf{,.$(date +%Y%m%d)}
# vi /etc/opt/remi/php80/php-fpm.d/www.conf
下記の通り調整
-----------------------------------------------------------------------------
listen = 127.0.0.1:9000
;listen.owner = nobody
;listen.group = nobody
;listen.mode = 0660
-----------------------------------------------------------------------------
↓
-----------------------------------------------------------------------------
listen = /var/run/php80-php-fpm/php80-php-fpm.sock
listen.owner = apache
listen.group = apache
listen.mode = 0660
-----------------------------------------------------------------------------
# diff /etc/opt/remi/php80/php-fpm.d/www.conf{,.$(date +%Y%m%d)}
```

#### 1-3-4. PHP-FPMプロセス関連ファイル調整
```
# mkdir /var/run/php80-php-fpm/

# vi /usr/lib/tmpfiles.d/varrun.conf
-----------------------------------------------------------------------------
#Type  Path                    Mode  UID   GID   Age  Argument
d      /var/run/php80-php-fpm  0755  root  root  -
-----------------------------------------------------------------------------
# cat /usr/lib/tmpfiles.d/varrun.conf
```

#### 1-3-5. PHP-FPM起動と自動起動有効化
```
# systemctl status php80-php-fpm.service
# systemctl enable --now php80-php-fpm.service
# systemctl status php80-php-fpm.service
# systemctl is-enabled php80-php-fpm.service
```

#### 1-3-6. 動作テスト②
```
# mkdir /var/www/vhosts/example.net/php80
# echo -e "<?php\nphpinfo();\n?>" > /var/www/vhosts/example.net/php80/phpinfo.php

# vi /var/www/vhosts/example.net/php80/.htaccess
※下記を追記
-----------------------------------------------------------------------------
<FilesMatch \.php$>
   SetHandler "proxy:unix:/var/run/php80-php-fpm/php80-php-fpm.sock|fcgi://localhost"
</FilesMatch>
-----------------------------------------------------------------------------
```

・ブラウザより、実行しているPHPが「PHP Version 8.0」であることを確認
https://example.net/php80/phpinfo.php

・ブラウザより、実行しているPHPが「PHP Version 7.3」であることを確認
https://example.net/php73/phpinfo.php

※利用した phpinfo.php は削除しておくこと