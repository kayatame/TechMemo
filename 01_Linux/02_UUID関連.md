## 1. UUIDの変更
UUIDを変更する場合は下記コマンドにて実行可能

```
# xfs_admin -U generate /dev/nvme1n1p4
```

この場合はUUIDはランダムとなる  
自分で指定する場合は下記のマニュアルを参照

[Red Hat Documentation(永続的な命名属性の変更)](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/8/html/managing_storage_devices/proc_modifying-persistent-naming-attributes_assembly_overview-of-persistent-naming-attributes)


## 2. マウント時のUUID重複
バックアップなどから復元したディスクをアタッチした場合にみられる事象
```
# mount -t xfs /dev/nvme1n1p4 /data
mount: /data: wrong fs type, bad option, bad superblock on /dev/nvme1n1p4, missing codepage or helper program, or other error.
```
UUIDが重複した場合、下記コマンドにて重複を無視してマウントが可能
```
# mount -t xfs -o nouuid /dev/nvme1n1p4 /data
```
※恒久的にマウントする場合、重複している状態は推奨されないことに注意
