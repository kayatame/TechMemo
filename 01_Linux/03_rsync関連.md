## 1. 基本コマンド
### 1-1. sshポートを利用した場合
```
# rsync -e 'ssh -i server.key' -av -n /var/www/html/ user@192.168.0.1:/var/www/html/
```
### 1-2. sudo権限を使用する場合
```
# rsync -e 'ssh -i server.key' --rsync-path="sudo rsync" -av -n /var/www/html/ user@192.168.0.1:/var/www/html/
```

上記はドライランなので、実際に実行する場合には「-n」を削除する  
「-a」 オプションを指定した場合は、ユーザ名/グループ名で同期される ※root 権限で実行する必要がある 

## 2. rsyncデータ転送テスト
### 2-1. テストデータ
```
# du dump.sql
20495256        dump.sql
# du -sh dump.sql
20G     dump.sql
```

### 2-2. 転送結果
| | 開始時刻 | 終了時刻 | 所要時間 |
|:--|---|---|--:|
| ローカル環境 | 01:22:00  | 01:24:39 | 2分39秒 |
| グローバル環境 | 02:14:59 | 02:20:28 | 5分29秒 |


## 3. 注意点
### 3-1. 移行元がディレクトリの場合末尾に「/」をつけないとそのディレクトリのまま移行される
#### 3-1-1. 末尾に「/」がない場合
**結論 : ディレクトリが転送される**
```
# rsync -av /var/www /home/user/
# ls -l /home/user/
   total 0
   drwxr-xr-x. 4 root root 33 Mar  8 01:56 www
```

#### 3-1-2. 末尾に「/」がある場合
**結論 : ディレクトリ配下が転送される**
```
# rsync -av /var/www/ /home/user/
# ls -l /home/user/
total 0
drwxr-xr-x. 2 root root  6 Jul 20  2023 cgi-bin
drwxr-xr-x. 2 root root 25 Mar  8 01:56 html
```
移行先と移行元のユーザグループには注意しておくこと

### 3-2. 移行先サーバにはユーザ/グループが存在しない場合
**結論 : UID/GIDがそのまま表示される**  
**同期元/同期先ともに同ユーザ/グループが存在する場合は問題なく移行が可能**

#### 3-2-1. 同期元
```
# ls -l rsync_test.txt
-rw-r--r--. 1 testuser testuser 11  Mar  8 04:15 rsync_test.txt
# rsync -e 'ssh -i new-server.key' -av ./rsync_test.txt root@***.***.***.***:/var/www/html/
```

#### 3-2-2. 同期先
```
# id testuser
id: ‘testuser’: no such user
# ll rsync_test.txt
-rw-r--r--. 1 1002 1002 11  Mar  8 04:15 rsync_test.txt
```

### 3-3. 移行先サーバにユーザ/グループが存在しないが、uidとgidに紐づくユーザとグループが存在する場合
**結論 : 移行先のUID/GIDに合わせて設定されてしまう**

#### 3-3-1. 同期元
```
# id testuser
uid=1002(testuser) gid=1002(testuser) groups=1002(testuser)
# ls -l rsync_test.txt
-rw-r--r--. 1 testuser testuser 11  Mar  8 04:15 rsync_test.txt
# rsync -e 'ssh -i new-server.key' -av ./rsync_test.txt root@***.***.***.***:/var/www/html/
```

#### 3-3-2. 同期先
```
# id test2user
uid=1002(test2user) gid=1002(test2user) groups=1002(test2user)
# ll /var/www/html/
rsync_test.txt -rw-r--r--. 1 test2user test2user 11  Mar  8 04:15 rsync_test.txt
```