- [ ] 123
- [x] 123
# MongoDB was attacked first in my life
## 緣由
> *人生第一次真的被駭客攻擊，真的需要好好紀錄一下*

2018年2月2日，一個寒冷的夜晚，突然老闆密我，正在Run的一個網站失常，我一進Server看server Log，不得了，資料全被刪了...😱😱😱
```nginx
0|server   | { error: 'NO THE Collection set' }
0|server   | { error: 'NO THE Collection set' }
0|server   | { error: 'NO THE Collection set' }
0|server   | { error: 'NO THE Collection set' }
...
```
經過一番詢問之後，確定相關人士都沒有動伺服器，只能懷疑被攻擊，畢竟，我貪圖方便直接**在防火牆開Mongodb的27017 port**外加**DB沒有設定auth**....
所以世界上任何人都可以簡單的IP:27017連上MongoDB然後為所欲為。

果然看到Mongod的log記錄:
```nginx
2018-01-30T03:59:17.385+0000 I NETWORK  [thread1] connection accepted from 207.226.143.128:43072
2018-01-30T03:59:17.428+0000 I COMMAND  [conn446] dropDatabase admin starting
2018-01-30T03:59:17.475+0000 I COMMAND  [conn446] dropDatabase admin finished
2018-01-30T03:59:17.475+0000 I COMMAND  [conn446] setting featureCompatibilityVersion to 3.2
2018-01-30T03:59:17.505+0000 I -        [conn446] end connection 207.226.143.128:43072 (6 connections now open)
2018-01-30T03:59:17.565+0000 I NETWORK  [thread1] connection accepted from 207.226.143.128:43076
2018-01-30T03:59:17.609+0000 I COMMAND  [conn447] dropDatabase test starting
2018-01-30T03:59:17.616+0000 I COMMAND  [conn447] dropDatabase test finished
2018-01-30T03:59:17.647+0000 I -        [conn447] end connection 207.226.143.128:43076 (6 connections now open)
2018-01-30T03:59:17.709+0000 I NETWORK  [thread1] connection accepted from 207.226.143.128:43078
2018-01-30T03:59:17.874+0000 I COMMAND  [conn448] command Warning.Readme command: 
insert { insert: "Readme", documents: [ 
	{ 
		_id: ObjectId('5a6fed95b5fdec4f33311aad'),
		BitCoin: "1FwSLsSJPVrWEwX3aQiHr1JqdcQA12M3ro",
		eMail: "mongodb@tfwno.gf", Exchange: "https://localbitcoins.com",
		Solution: "Your Database is downloaded and backed up on our secured servers."
		 " To recover your lost data: Send 0.1 BTC to our BitCoin Address and Contact us by eMa..." 
	} 
]
```
直接drop整個資料庫，最後只留下Bitcoin得帳戶所要0.1比特幣，當時查一下匯率...五萬多新台幣....
夠狠! 你行! 🖕
不過還好該網站的資料庫還在測試階段，最近備份也在不久之前，所以簡單作法是直接restore
`mongorestore -h 127.0.0.1 -d my-mongo-new --directoryperdb ./mongo-backup/my-mongo`
但國中導師教我一個觀念
> **危機就是轉機**

**所以必須重中學到教訓，加強資安措施與增加備援機制**
## 環境
|  Object  |  Version|
 ----------| ---------|
|  Ubuntu |  14.04|
|  Nodejs  |  9.3|
| MongoDB | 3.4.10|
|Mongoose| 4.13.10|
# 處理步驟
## Step 1: MongoDB authorization

1. 預設情況直接開啟Mongod服務會是以下設定 `mongod --port 27017 --dbpath /data/db1` ，然後連接只需`mongo --port 27017` or `mongo` 。
2. 選擇使用admin資料庫並創一個有帳密的使用者
```js
use admin
db.createUser(
  {
    user: "DBowner",
    pwd: "abc123",
    roles: [ { role: "DBowner", db: "admin" } ]
  }
)
```
其中`DBowner` 為Super User，[官方文件](https://docs.mongodb.com/manual/reference/built-in-roles/#superuser-roles)上說明可以授權(auth)任何user到任何DB，包括自己。
~所以我覺得這會是一個後患，但修復要緊~

3.  重啟Mongod
`mongod --auth` or `sudo service mongod restart`
> NOTE:
> 因為後者無法pass --auth參數，所以得多做一個步驟: 
> `sudo vim /etc/mongod.conf`
> 找到並修改成
> ```git
> - #security:
> + security:
> +    authorization: enabled
>   ```
> 
4.  現在開始要操作admin的話都要pass帳密了
```
mongo --port 27017 -u "myUserAdmin" -p "abc123" --authenticationDatabase "admin"
```
或是先連上Mongod再進行驗證
```javascript
$ mongo
> use admin
> db.auth("myUserAdmin", "abc123" )
```
## Step 2: 備份回復 Restore Database
預設dump出來的資料會放在`/data/dump` 下，只要到該路徑並使用`mongorestore` 
```
$ cd /data/dump
$ mongorestore -h localhost -d admin -u myUserAdmin -p abc123 --authenticationDatabase admin ./test
```
-h
: host IP

-d
: Database Name

-u, -p
: user name and password

-\-authenticationDatabase
: 該user是在哪個DB create的

./test
: 備份的檔案位置(.bson)，當初備份test的資料所以資料夾名稱是test
## Step 3: 更新Mongoose連線設定 Update Mongoose Connection Setting
1.  首先保險起見，給API server一個獨立的user，而且權限縮小，只保留讀寫該DB功能
```
> use admin
> db.createUser({
 user:"node",
 pwd:"node",
 roles:[{role:"readWrite", db:"admin"}]
})
```
2.  到nodejs設定連接MongoDB的地方
```github
- mongoose.connection.openUri('mongodb://localhost/test');
+ mongoose.connection.openUri('mongodb://node:node@localhost/admin');
```
3.  重啟整個服務
```
$ pm2 stop all
$ pm2 start --interpreter ./node_modules/.bin/babel-node server.js
```
![enter image description here](https://lh3.googleusercontent.com/RqExzMZB9dQ2YzWaFaYfFNZ7fTfNf0yEP2ypvmTBEEt9Jv-Y5LPt8AT-VBMOSzMrUNL1JXKsIdwpTmkvpBR6f7AluuyyOfl1HL1I-A2MCGaS0Q5gXx-wOX35aX29xLgKVqXG7gPdUrM59Dmq6jo_pjvtA-NIPfhaLT_WcREQE_-qq5BUSHwTaBWFMeQptyvk5DCco4xJ9m2q1ZWh8svnQjOHR7n8obr3qYf6sQZiBxLp_zFH9wNJx1ZAAYW26eE5VhQFkuZ8FMhElgnvLL6ONl5KsHFgWyGhtUdK8qRkA2Q4vC2z1dpm9lP-WbtWqEArNUBHtN1XljtTjptmiyr2rF5qYzwKifW3jHhL6KQPtlShO89EMQYzx_REt8I5lU1EiGW5DBYcwtAs2C3ZbfQw7ZF6BZzWBTPgXlXRFQgLJ-9jtTgcqNDnc1lsogx0kX-EF7Aehh8ipnDj7Nd9p6r5E6CR5YgmBAhC1lEo6wKbBRrYlM5j1ZOkTTVCSZe-UNev7aYtY_QTcn8IyFpHWdBzt6yhluWWXWYiCspMTw2xgUIDG_DJn_lLJ8TP09yqax6e1TMTbAZuzyxme5U8F_DoQL5yoJwgBIhysdEtjlAhv97S_nZ-y3kccfcC0zqc0en1FJBu8_w0EmOV5sK0mIwXf2K38YPmj48r_A=w1620-h957-no)
資料回來了感動QQ
## Step 4: 設定每日備份資料 Daily DB Backup
不管什麼原因，為了防止以後資料再度消失，備份很重要
cron.daily會在每天的凌晨4:02執行該路徑底下的shell script，基本上四點是網站最冷門時段，所以非常符合需求
1. `sudo vim /etc/cron.daily`
2.  [借用此教學的內容](https://brickyang.github.io/2017/03/02/Linux-%E8%87%AA%E5%8A%A8%E5%A4%87%E4%BB%BD-MongoDB/)，依照需求修改即可
```bash
#!/bin/sh
DUMP=mongodump
OUT_DIR=/data/backup/mongod/tmp   // 备份文件临时目录
TAR_DIR=/data/backup/mongod       // 备份文件正式目录
DATE=`date +%Y_%m_%d_%H_%M_%S`    // 备份文件将以备份时间保存
DB_USER=<USER>                    // 数据库操作员
DB_PASS=<PASSWORD>                // 数据库操作员密码
DAYS=14                           // 保留最新14天的备份
TAR_BAK="mongod_bak_$DATE.tar.gz" // 备份文件命名格式
cd $OUT_DIR                       // 创建文件夹
rm -rf $OUT_DIR/*                 // 清空临时目录
mkdir -p $OUT_DIR/$DATE           // 创建本次备份文件夹
$DUMP -u $DB_USER -p $DB_PASS -o $OUT_DIR/$DATE  // 执行备份命令
tar -zcvf $TAR_DIR/$TAR_BAK $OUT_DIR/$DATE       // 将备份文件打包放入正式目录
find $TAR_DIR/ -mtime +$DAYS -delete             // 删除14天前的旧备份
```

