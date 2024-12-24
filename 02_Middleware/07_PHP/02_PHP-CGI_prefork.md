## 1.prefork設定手順
### 1-1. MPM調整
```
# httpd -V | grep "MPM"
# grep -v "^\s*#\|^\s*$" /etc/httpd/conf.modules.d/00-mpm.conf
# vi /etc/httpd/conf.modules.d/00-mpm.conf
-----------------------------------------------------------------------------
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
#LoadModule mpm_event_module modules/mod_mpm_event.so
-----------------------------------------------------------------------------
# grep -v "^\s*#\|^\s*$" /etc/httpd/conf.modules.d/00-mpm.conf

# httpd -t
# systemctl restart httpd
```

RHEL8系以降、AppStreamでは mod_php は廃止しているので、prefork かつ mod_php で対応する場合はRemiからインストールする必要がある

> Apache HTTP サーバーで使用するために PHP に提供されている mod_php モジュールは削除されました。  
> Apache HTTP サーバーで使用するために PHP に付属している mod_php モジュールは、RHEL 9 では使用できなくなりました。

※なおAppStreamからインストールしたPHPでもprefork設定は可能であるがPHP-FPMの起動が必要になる

[Red Hat Documentation(RHEL 9 の採用における考慮事項)](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/9/html-single/considerations_in_adopting_rhel_9/index#ref_removed-functionality-identity-management_assembly_identity-management)

### 1-2. Remiリポジトリインストール
```
# dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
# dnf repolist all | grep "remi"

前提としてEPELがインストールされている必要がある
# dnf install epel-release
```
[Remi公式サイト](https://rpms.remirepo.net/)

### 1-3. モジュール有効化
```
# dnf module enable php:remi-8.0
```

### 1-4. パッケージインストール
```
# dnf install php php-fpm php-cli php-devel php-common php-pear php-mysqlnd
# php -v
# php -m
```

### 1-5. php.ini調整
```
# cp -ip /etc/php.ini{,.org}
# ll /etc/php.ini{,.org}
# vi /etc/php.ini
------------------------------------------------
expose_php = Off
max_execution_time = 60
memory_limit = 256M
html_errors = Off
post_max_size = 128M
upload_max_filesize = 128M
date.timezone = 'Asia/Tokyo'
mbstring.language = Japanese
mbstring.internal_encoding = UTF-8
------------------------------------------------
# diff /etc/php.ini{,.org}
```

### 1-6. 反映
```
# httpd -t
# systemctl restart httpd
# httpd -M | grep "php"
```

### 1-7. ユーザ権限調整
```
# usermod -g apache user
```

### 1-8. phpinfo確認
```
# echo -e "<?php\nphpinfo();\n?>" /var/www/vhosts/example.com/phpinfo.php

確認後は削除しておくこと
# rm /var/www/vhosts/example.com/phpinfo.php
```

### 1-9. PHP-FPM自動起動停止
```
# systemctl stop php-fpm
# systemctl disable php-fpm
# mv /etc/systemd/system/httpd.service.d/php-fpm.conf{,.bk}
```

### 1-10. EPELリポジトリ無効化
```
# dnf repolist all | grep -i epel

有効化になっているリポジトリの設定ファイル内の「enabled」を「0」に変更
# cat /etc/yum.repos.d/epel.repo
# vi /etc/yum.repos.d/epel.repo
------------------
enabled=0
------------------
# cat /etc/yum.repos.d/epel.repo
```