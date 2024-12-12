## 1. OOM Killer発生時のダンプ
OOM Killerが発生した場合にログへプロセス状況を出力するには下記設定が必要
```
# sysctl vm.oom_dump_tasks
vm.oom_dump_tasks = 1

# cat /proc/sys/vm/oom_dump_tasks
1
```
[OOM Killer に関連するカーネルパラメータまとめ](https://reboooot.net/post/kernel-params-related-to-oom-killer/)
