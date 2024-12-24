## 1. php-devel
### 1-1. php-develとは
そもそも「devel」とは、「development」の略  
PHP の拡張モジュールを作成したり、PHP のソースコードをコンパイルするために必要なファイルが含まれているパッケージ

### 1-2. インストール方法(AlmaLinux8)
インストールには libedit-devel が必要とのこと
```
# dnf install php-devel
Last metadata expiration check: 0:08:11 ago on Fri 23 Aug 2024 04:21:06 AM JST.
Error:
 Problem: cannot install the best candidate for the job
  - nothing provides libedit-devel(x86-64) needed by php-devel-7.3.33-15.el8.remi.x86_64 from remi-modular
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)

# dnf --enablerepo=powertools install libedit-devel
```
