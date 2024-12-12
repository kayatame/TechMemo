## 1. SMTP認証確認方法
下記の手順でSMTP認証のログイン確認が可能
```
$ printf "%s\0%s\0%s" アカウント名 アカウント名 パスワード | openssl base64 -e | tr -d '\n'; echo
****************************************************************************

$ telnet mail.example.com 25 
Trying ***.***.+***.***...
Connected to  mail.example.com.
Escape character is '^]'.
220  ************* SMTP unknown
EHLO example.com                                                                             # 入力箇所①
～
AUTH PLAIN ****************************************************************************      # 入力箇所②
235 2.7.0 Authentication successful
quit
221 2.0.0 Bye
Connection closed by foreign host.
```