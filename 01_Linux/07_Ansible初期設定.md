## 1. 初期設定
### 1-1. Ansibleインストール(コントロールノード)
```
[user@Control ~]$ sudo dnf install ansible-core
[user@Control ~]$ ansible --version
```

### 1-2. 秘密鍵作成(コントロールノード)
```
[user@Control ~]$ ssh-keygen -t rsa
```

### 1-3. ユーザ作成(ターゲットノード)
```
[root@Target ~]# useradd 実行ユーザ
[root@Target ~]# passwd 実行ユーザ
[root@Target ~]# usermod -aG wheel 実行ユーザ
```

### 1-4. ターゲットノードへ鍵複製(コントロールノード)
```
[user@Control ~]$ ssh-copy-id -o StrictHostKeyChecking=no -i $HOME/.ssh/id_rsa.pub 実行ユーザ@ターゲットノードIP
```
※SSH接続する場合には下記のコマンド
```
[user@Control ~]$ ssh-copy-id -o StrictHostKeyChecking=no -i $HOME/.ssh/id_rsa.pub -o IdentityFile=秘密鍵 実行ユーザ@ターゲットノードIP
```

## 2. Ansibleテスト実行(コントロールノード)
```
[user@Control Ansible]$ vim inventory.ini
---------------------
[test_servers]
ターゲットノードIP
---------------------

[user@Control Ansible]$ ansible -u 実行ユーザ -m ping -i inventory.ini test_servers
ターゲットノードIP | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```