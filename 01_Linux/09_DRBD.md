## 1. DRBD設定手順
### 1-1. DRBD用ディスク設定(Master/Slave)
※ディスクを認識させる前に実施すること
```
# gdisk /dev/nvme1n1
-----------------------------------------------------------------------------
～
Command (? for help): n
Partition number (1-128, default 1): 1
First sector (34-20971486, default = 2048) or {+-}size{KMGTP}: 2048
Last sector (2048-20971486, default = 20971486) or {+-}size{KMGTP}: 20971486
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): Enter押下のみ
～
Command (? for help): p
～
Command (? for help): w
～
Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/nvme2n1.
The operation has completed successfully.
-----------------------------------------------------------------------------
# partprobe
# mkdir /drbd
# ll -d /drbd
```

### 1-2. パッケージインストール(Master/Slave)
```
# dnf install elrepo-release

# vi /etc/yum.repos.d/elrepo.repo
-----------------------------------------------------------------------------
変更前 : enabled=1
変更前 : enabled=0
-----------------------------------------------------------------------------
# grep "^enable" /etc/yum.repos.d/elrepo.repo

# dnf --enablerepo=elrepo install kmod-drbd9x drbd9x-utils
# reboot
```

### 1-3. DRBD設定調整(Master/Slave)
```
# cp -ip /etc/drbd.d/global_common.conf{,.org}
# vi /etc/drbd.d/global_common.conf
-----------------------------------------------------------------------------
global {
        usage-count no;
        udev-always-use-vnr; # treat implicit the same as explicit volumes
}
common {
        handlers {
        }
        startup {
        }
        options {
        }
        disk {
                resync-rate 20M;
        }
        net {
                protocol C;
        }
-----------------------------------------------------------------------------
# grep -v "^\s*#\|^\s*$" /etc/drbd.d/global_common.conf
```

### 1-4. リソース・ボリュームの定義(Master/Slave)
```
# vi /etc/drbd.d/r0.res
-----------------------------------------------------------------------------
resource r0 {
  device /dev/drbd0;
  disk /dev/nvme1n1;             # DRBD用に接続したディスク
  on master {
     node-id 0;
     address  192.168.0.1:7788;
     meta-disk internal;
  }
  on slave {
     node-id 1;
     address 192.168.0.2:7788;
     meta-disk internal;
  }
}
-----------------------------------------------------------------------------

# vi /etc/hosts
-----------------------------------------------------------------------------
master 192.168.0.1
slave  192.168.0.2
-----------------------------------------------------------------------------
```

### 1-5. メタデータ作成 / DRBD起動(Master/Slave)
※DB1/DB2で同時に起動する
```
# drbdadm create-md r0
# systemctl start drbd
```

### 1-6. プライマリの調整(Master)
```
# drbdadm primary --force r0
# drbdadm status r0

設定後のステータス状況
※少し時間が必要になる  
-----------------------------------------------------------------------------
・Master
# drbdadm status r0
r0 role:Primary
  disk:UpToDate
  slave role:Secondary
    peer-disk:UpToDate
-----------------------------------------------------------------------------
・Slave
# drbdadm status r0
r0 role:Secondary
  disk:UpToDate
  master role:Primary
    peer-disk:UpToDate
-----------------------------------------------------------------------------
```

### 1-7. /dev/drbd0をMySQLディスクとして利用するためフォーマット / mount(Master)
```
# mkfs -t ext4 /dev/drbd0
# mount /dev/drbd0 /drbd
# df -Th
```

### 1-8. マウントテスト(Master/Slave)
```
・Master
# echo "test" > /drbd/test.txt ; chmod 705 /drbd/test.txt
# ll /drbd/
# umount /drbd
# drbdadm secondary r0
# drbdadm status r0     # ここでSecondaryになっていることを確認

・Slave
# drbdadm primary r0
# drbdadm status r0      # ここでPrimaryになっていることを確認
# mount /dev/drbd0 /drbd
# ll /drbd
```
以降の作業を進めるために、MasterとSlaveを変えておく

### 1-9. MySQL関連設定
#### 1-9-1. MySQLインストール(Master/Slave)
　/dev/drbd0 が /data にマウントされている状態で進める
```
# dnf install https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
# dnf install mysql-community-server
# mysql -V

ダウンロード時にGPGキーのエラーが発生する場合は下記コマンドにて鍵をインストール
# rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
```
[MySQL 8.0 Reference Manual(Signature Checking Using RPM)](https://dev.mysql.com/doc/refman/8.0/en/checking-rpm-signature.html)


#### 1-9-2. MySQL初期設定(Master)
```
# mv /var/lib/mysql /drbd/mysql
# ln -s /drbd/mysql /var/lib/mysql

# systemctl start mysqld
# systemctl status mysqld

# systemctl stop mysqld
```

#### 1-9-3. MySQL初期設定(Slave)
```
# mv /var/lib/mysql /tmp/
# ln -s /drbd/mysql /var/lib/mysql
```

### 1-10. サービス停止とアンマウント(Master/Slave)
```
# umount /drbd
# systemctl stop drbd
```

### 1-11. Pacemaker+Corosync+Pcsdパッケージインストール(Master/Slave)
```
# dnf --enablerepo=highavailability install corosync pacemaker pcs

# grep "hacluster" /etc/passwd
# passwd hacluster

# systemctl start pcsd
```

### 1-12. pcsクラスタ設定(Master)
#### 1-12-1. corosyncの設定ファイルを記述
```
# vi /etc/corosync/corosync.conf
-----------------------------------------------------------------------------
totem {
   version: 2
   cluster_name: my_cluster      # 任意の名前で設定可能
   transport: knet
   crypto_cipher: aes256
   crypto_hash: sha256
}

nodelist {
   node {
       ring0_addr: 172.31.12.26  # 下記のホスト名に対応したプライベートIPを指定
       name: master              # /etc/hostsに追記したホスト名を指定
       nodeid: 1
   }

   node {
       ring0_addr: 172.31.3.31   # 下記のホスト名に対応したプライベートIPを指定
       name: slave               # 上記ともう一方の/etc/hostsに追記したホスト名を指定
       nodeid: 2
   }
}

quorum {
   provider: corosync_votequorum
   two_node: 1
}

logging {
   to_logfile: yes
   logfile: /var/log/cluster/corosync.log
   to_syslog: yes
   timestamp: on
}
-----------------------------------------------------------------------------
```

#### 1-12-2. クラスタ構成インスタンスの事前認証
```
# pcs cluster auth
Warning: Unable to read the known-hosts file: No such file or directory: '/var/lib/pcsd/known-hosts'
master: Not authorized
slave: Not authorized
Nodes to authorize: master, slave
Username: hacluster
Password:
master: Authorized
slave: Authorized
```

#### 1-12-3. クラスタのセットアップを行う
```
# pcs cluster setup my_cluster master addr=172.31.12.26 slave addr=172.31.3.31 --force
Warning: master: The host seems to be in a cluster already as cluster configuration files have been found on the host. If the host is not part of a cluster, run 'pcs cluster destroy' on host 'master' to remove those configuration files
Destroying cluster on hosts: 'master', 'slave'...
slave: Successfully destroyed cluster
master: Successfully destroyed cluster
Requesting remove 'pcsd settings' from 'master', 'slave'
master: successful removal of the file 'pcsd settings'
slave: successful removal of the file 'pcsd settings'
Sending 'corosync authkey', 'pacemaker authkey' to 'master', 'slave'
master: successful distribution of the file 'corosync authkey'
master: successful distribution of the file 'pacemaker authkey'
slave: successful distribution of the file 'corosync authkey'
slave: successful distribution of the file 'pacemaker authkey'
Sending 'corosync.conf' to 'master', 'slave'
master: successful distribution of the file 'corosync.conf'
slave: successful distribution of the file 'corosync.conf'
Cluster has been successfully set up.
```

#### 1-12-4. pcsクラスタの起動
```
# pcs cluster start --all
master: Starting Cluster...
slave: Starting Cluster...
```

#### 1-12-5. pcsクラスタの自動起動
```
# pcs cluster enable --all
master: Cluster Enabled
slave: Cluster Enabled
```

#### 1-12-6. STONITHの無効化
```
# pcs property set stonith-enabled=false
```

#### 1-12-7. pcsのステータス確認
```
# pcs status
Cluster name: my_cluster
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: slave (version 2.1.7-5.el9_4-0f7f88312) - partition with quorum
  * Last updated: Tue Aug  6 10:28:39 2024 on master
  * Last change:  Tue Aug  6 10:28:18 2024 by root via root on master
  * 2 nodes configured
  * 0 resource instances configured

Node List:
  * Online: [ master slave ]

Full List of Resources:
  * No resources

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/disabled
```

#### 1-12-8. Slaveに設定ファイルが同期されていることを確認(Slave)
```
# less /etc/corosync/corosync.conf
```

### 1-13. pcsリソース登録(Master)
#### 1-13-1. DRDB管理用リソースの登録
```
# pcs resource create drbd_r0 ocf:linbit:drbd drbd_resource=r0 op monitor interval=10s role=Master monitor interval=30s role=Slave
Deprecation Warning: Value 'Master' of option role is deprecated and should not be used, use 'Promoted' value instead
Deprecation Warning: Value 'Slave' of option role is deprecated and should not be used, use 'Unpromoted' value instead
```

#### 1-13-2. DRBD プライマリ/セカンダリ確認用リソース登録
```
# pcs resource promotable drbd_r0 master-max=1 master-node-max=1 clone-max=2 clone-node-max=1 notify=true
Deprecation Warning: configuring meta attributes without specifying the 'meta' keyword is deprecated and will be removed in a future release
```

#### 1-13-3. DRBDデバイスを/mnt/drbd0にマウントするリソースをmysqlリソースグループとして登録
　directoryの指定に注意　自身でDRBD領域として設定したディレクトリを指定
```
# pcs resource create fs_mysql ocf:heartbeat:Filesystem device=/dev/drbd0 directory=/drbd fstype=ext4 --group mysql
Deprecation Warning: Using '--group' is deprecated and will be replaced with 'group' in a future release. Specify --future to switch to the future behavior.
```

#### 1-13-4. drbd_r0-clone と mysql リソースグループの一連の処理を同一サーバで実行するよう colocation制約の登録
```
# pcs constraint colocation add mysql with drbd_r0-clone INFINITY with-rsc-role=Master
```

#### 1-13-5. DRBDがMasterのステータスになってから、mysqlリソースグループの処理を始めるorder制約を登録
```
# pcs constraint order promote drbd_r0-clone then start mysql
Adding drbd_r0-clone mysql (kind: Mandatory) (Options: first-action=promote then-action=start)
```

#### 1-13-6. fs_mysqlリソース登録時点で出ているエラーを一度クリア
```
# pcs resource cleanup
Cleaned up all resources on all nodes
Waiting for 3 replies from the controller
... got reply
... got reply
... got reply (done)
```

#### 1-13-7. MySQL管理用リソースを mysqlリソースグループに登録
```
# pcs resource create mysqld systemd:mysqld --group mysql
Deprecation Warning: Using '--group' is deprecated and will be replaced with 'group' in a future release. Specify --future to switch to the future behavior.
```

#### 1-13-8. VIP管理用リソースを mysqlリソースグループに登録
※vipはMaster/Slaveで設定したプライベートIP以外を指定  
　ネットマスクとNICは基本的には /24 と eth0(既存ローカルIPが付与されているNIC) を選択しておく
```
# pcs resource create VirtualIP ocf:heartbeat:IPaddr2 ip=172.31.3.12 cidr_netmask=24 nic="eth0" --group mysql
Deprecation Warning: Using '--group' is deprecated and will be replaced with 'group' in a future release. Specify --future to switch to the future behavior.
```

#### 1-13-9. セットアップ完了を確認
```
# pcs status
Cluster name: my_cluster
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: master (version 2.1.7-5.el9_4-0f7f88312) - partition with quorum
  * Last updated: Tue Aug  6 11:33:13 2024 on master
  * Last change:  Tue Aug  6 11:29:39 2024 by hacluster via hacluster on slave
  * 2 nodes configured
  * 5 resource instances configured

Node List:
  * Online: [ master slave ]

Full List of Resources:
  * Clone Set: drbd_r0-clone [drbd_r0] (promotable):
    * Promoted: [ master ]
    * Unpromoted: [ slave ]
  * Resource Group: mysql:
    * fs_mysql  (ocf:heartbeat:Filesystem):      Started master
    * mysqld    (systemd:mysqld):        Started master
    * VirtualIP (ocf:heartbeat:IPaddr2):         Started master

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: inactive/disabled
```

### 1-14. フェイルオーバテスト
Primaryを再起動してSecondary でMySQLが起動しておりVIPが付与されていることを確認
```
・Master
# reboot

・Slave
# pcs status
# systemctl status mysqld
# ip a
```