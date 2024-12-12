## 1. MySQLTuner
### 1-1. 公式リポジトリ
[Github公式リポジトリ(MySQLTuner)](https://github.com/major/MySQLTuner-perl)

### 1-2. インストール
```
# cd /home/your-assist/mysqltuner/
# wget --no-check-certificate http://mysqltuner.pl/ -O mysqltuner.pl
# wget --no-check-certificate https://raw.githubusercontent.com/major/MySQLTuner-perl/master/basic_passwords.txt -O basic_passwords.txt
# wget --no-check-certificate https://raw.githubusercontent.com/major/MySQLTuner-perl/master/vulnerabilities.csv -O vulnerabilities.csv
```

### 1-3. 実行
```
# perl mysqltuner.pl --host localhost
```