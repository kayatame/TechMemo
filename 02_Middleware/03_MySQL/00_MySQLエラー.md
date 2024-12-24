## 1. Access denied for user
### 1-1. 対象エラー
```
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
```

### 1-2. 原因
諸々  

### 1-3. 対応
インストール直後の場合、パスワードを入力してログインする必要がある  
パスワード確認方法 : grep "temporary password" /var/log/mysqld.log  


## 2. InnoDB: Cannot allocate memory for the buffer pool
### 2-1. 対象エラー
```
[Note] InnoDB: Initializing buffer pool, size = 64.0G
InnoDB: mmap(1098907648 bytes) failed; errno 12
[ERROR] InnoDB: Cannot allocate memory for the buffer pool
```

### 2-2. 原因
サーバの搭載メモリ以上のメモリをバッファプールへ保存しようとしていた  

### 2-3. 対応
「innodb_buffer_pool_size」の調整  
※搭載メモリ以下の数値とすること  


## 3. InnoDB: Cannot set log file
### 3-1. 対象エラー
```
[Note] InnoDB: Setting log file ./ib_logfile101 size to 16384 MB
InnoDB: Progress in MB: 100 200 300 400 500 600 700 800 900 1000 1100 1200 1300 1400 1500 1600 1700 1800 1900 2000 2100 2200 2300 2400 2500 2600 2700 2800 2900 3000 3100 3200 3300 3400 3500
 3600 3700 3800 3900 40002024-05-25 17:37:48 7f0948b207802024-05-25 17:37:48 8724 [ERROR] InnoDB: Failure of system call pwrite(). Operating system error number is 28.
InnoDB: Error number 28 means 'No space left on device'.
InnoDB: Some operating system error numbers are described at
InnoDB: http://dev.mysql.com/doc/refman/5.6/en/operating-system-error-codes.html
[ERROR] InnoDB: Cannot set log file ./ib_logfile101 to size 16384 MB
```

### 3-2. 原因  
ディスク容量よりも大きいサイズの「ib_logfile(InnoDBのログファイル)」を作成しようとしていた  

### 3-3. 対策  
「innodb_log_file_size」の調整  


## 4. Performance schema disabled
### 4-1. 対象エラー
```
[Warning] Buffered warning: Performance schema disabled (reason: init failed).
```

### 4-2. 原因 
メモリ不足  ※他にも要因は考えられる  

### 4-3. 対応
メモリ不足の場合 : スワップ領域の作成等  


## 5. Incorrect string value
### 5-1. 対象エラー
```
ERROR 1366 (HY000): Incorrect string value: '\xE5\x95\x86\xE5\x93\x81...' for column 'name' at row 1
```

### 5-2. 原因 
異なる文字コードのデータを挿入  
対象テーブルやデータベースの設定文字コードを変更する  

### 5-3. 対応
データベース : SHOW CREATE TABLE テーブル名;  
テーブル     : SHOW CREATE DATABASE データベース名;  

[MySQLの文字コードをutf8mb4に変更](https://qiita.com/decoch/items/bfa125ae45c16811536a)

## 6. Upgrade is not supported after a crash or shutdown
### 6-1. 対象エラー
```
[ERROR] [MY-012526] [InnoDB] Upgrade is not supported after a crash or shutdown with innodb_fast_shutdown = 2. This redo log was created with MySQL 5.7.44, and it appears logically non empty. Please follow the instructions at http://dev.mysql.com/doc/refman/8.0/en/upgrading.html
```

### 6-2. 原因  
redoログファイルは残存している

### 6-3. 対応
redoログファイルの削除
```
# rm /var/lib/mysql/ib_logfile0 /var/lib/mysql/ib_logfile1
```