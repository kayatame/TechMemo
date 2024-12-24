## 1. mysqldumpテスト
### 1-1. 20GBデータのmysqldump(エクスポート)
```
# mysqldump --master-data=2 --all-databases -u root > dump.sql

# du -sh dump.sql
20G     dump.sql
```

| 開始時刻 | 終了時刻 | 所要時間 |
| --- | --- | --- |
| 00:50:00 | 00:59:52 | 約10分 |

※80GBのダンプデータをmysqldumpで出力した場合、65Gになったので一概に同じとは言えない

### 1-2. インポート
| 開始時刻 | 終了時刻 | 所要時間 |
| --- | --- | --- |
| 02:45:00 | 03:51:47 | 1時間6分47秒 |
