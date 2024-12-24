## 1. SELinux起動時のポート設定
### 1-1. SELinuxで許可されているポート確認
```
# semanage port -l | grep 20443
```

### 1-2. ファイルラベリングルールを追加
```
# semanage port -a -t http_port_t -p tcp 20443
```

### 1-3. 再度確認
```
# semanage port -l | grep 20443
http_port_t                    tcp      20443, 80, 81, 443, 488, 8008, 8009, 8443, 9000
```

[とほほのSELinux入門](https://www.tohoho-web.com/ex/selinux.html#semanage)