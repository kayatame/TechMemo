## 1. 初期設定
```
useradd -G wheel ansi
passwd ansi

mkdir /home/ansi/.ssh
cp -i /home/ec2-user/.ssh/authorized_keys /home/ansi/.ssh/

chown -R ansi: /home/ansi/.ssh
```
```
$ ansible-playbook -i ./inventory.ini ./default_settings/main.yml --private-key /home/kayatame/.ssh/default_key.pem -K
```