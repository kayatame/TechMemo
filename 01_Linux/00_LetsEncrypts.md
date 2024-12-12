## 1. パッケージインストール
```
# dnf install certbot
# which certbot
```
※certbot をインストールする際にはEPELが必要

```
# dnf install epel-release
```

## 2. 事前確認
### 2-1. DNS確認
```
# dig +noall +ans {,www.}example.com
```

### 2-2. 接続確認
80番ポートを解放していることを確認


## 3. 証明書発行
### 3-1. ドライラン
```
# certbot certonly --webroot -w /var/www/vhosts/example.com -d example.com -d www.example.com --dry-run
```

### 3-2. 発行
「-w」ではドキュメントルートを指定
```
# certbot certonly --webroot -w /var/www/vhosts/example.com -d example.com -d www.example.com
=============================================================================================
# メールアドレスを登録
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): root@example.com

# 利用条件に同意する
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.4-April-3-2024.pdf. You must agree in
order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

# 非営利団体 Electronic Frontier Foundation にもメールアドレスを登録するか否か
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
Account registered.
Requesting a certificate for example.com and www.example.com

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/example.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/example.com/privkey.pem
This certificate expires on 2024-10-30.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
=============================================================================================

# ll /etc/letsencrypt/live/example.com/*
cert.pem       # SSLサーバー証明書 (公開鍵含む)
chain.pem      # 中間証明書
fullchain.pem  # cert.pem と chain.pem が結合されたファイル
privkey.pem    # 公開鍵に対する秘密鍵
```