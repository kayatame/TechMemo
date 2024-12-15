## 1. SFTP関連設定
### 1-1. sshd_config調整
```
# cp -ip /etc/ssh/sshd_config{,.org}
# vi /etc/ssh/sshd_config
----------------------------------------------------------------------------------------
PasswordAuthentication no
PermitRootLogin no
Subsystem     sftp    /usr/libexec/openssh/sftp-server -u 002 -f local5 -l VERBOSE
----------------------------------------------------------------------------------------
# diff /etc/ssh/sshd_config{,.org}

# sshd -t
# systemctl status sshd
# systemctl restart sshd
```
[Red Hat Documentation (OpenSSH のセキュリティーの強化)](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/making-openssh-more-secure_assembly_using-secure-communications-between-two-systems-with-openssh)


### 1-2. SFTPロギング設定
```
# mkdir /var/log/sftp/
# vi /etc/rsyslog.d/sftp.conf
----------------------------------------------------
local5.*        /var/log/sftp/sftp.log
----------------------------------------------------
# systemctl restart rsyslog

# cp -ip /etc/logrotate.d/rsyslog /home/user/logrotate_rsyslog.org
# vi /etc/logrotate.d/rsyslog
---------------------------------------------------
/var/log/sftp/sftp.log   #ローテート対象に追加
---------------------------------------------------
```