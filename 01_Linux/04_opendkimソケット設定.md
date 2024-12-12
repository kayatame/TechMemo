## 1. OpenDKIMソケット設定
Socket の設定によりUNIXソケットかTCPソケットのどちらでListenするか決まる  

### 1-1. UNIXソケット
```
# grep "Socket" /etc/opendkim.conf
#Socket inet:8891@localhost
Socket local:/run/opendkim/opendkim.sock

# netstat -lx | grep "opendkim"
unix  2      [ ACC ]     STREAM     LISTENING     154515   /run/opendkim/opendkim.sock
```

### 1-2. TCPソケット
```
# grep "Socket" /etc/opendkim.conf
Socket inet:8891@localhost
#Socket local:/run/opendkim/opendkim.sock
 
# netstat -tulpn | grep "opendkim"
tcp        0      0 127.0.0.1:8891          0.0.0.0:*               LISTEN      16081/opendkim
```