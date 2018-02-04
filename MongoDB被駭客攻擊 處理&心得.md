---
# Add custom properties for the current file here
# to override the default properties.
title: MongoDB was attacked first in my life
author: zsh

---

<h1 id="mongodb-was-attacked-first-in-my-life">MongoDB was attacked first in my life</h1>
<h2 id="緣由">緣由</h2>
<blockquote>
<p><em>人生第一次真的被駭客攻擊，真的需要好好紀錄一下</em></p>
</blockquote>
<p>2018年2月2日，一個寒冷的夜晚，突然老闆密我，正在Run的一個網站失常，我一進Server看server Log，不得了，資料全被刪了…😱😱😱</p>
<pre class=" language-nginx"><code class="prism  language-nginx"><span class="token number">0</span><span class="token operator">|</span><span class="token keyword">server</span>   <span class="token operator">|</span> <span class="token punctuation">{</span> error<span class="token punctuation">:</span> <span class="token string">'NO THE Collection set'</span> <span class="token punctuation">}</span>
<span class="token number">0</span><span class="token operator">|</span><span class="token keyword">server</span>   <span class="token operator">|</span> <span class="token punctuation">{</span> error<span class="token punctuation">:</span> <span class="token string">'NO THE Collection set'</span> <span class="token punctuation">}</span>
<span class="token number">0</span><span class="token operator">|</span><span class="token keyword">server</span>   <span class="token operator">|</span> <span class="token punctuation">{</span> error<span class="token punctuation">:</span> <span class="token string">'NO THE Collection set'</span> <span class="token punctuation">}</span>
<span class="token number">0</span><span class="token operator">|</span><span class="token keyword">server</span>   <span class="token operator">|</span> <span class="token punctuation">{</span> error<span class="token punctuation">:</span> <span class="token string">'NO THE Collection set'</span> <span class="token punctuation">}</span>
<span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
</code></pre>
<p>經過一番詢問之後，確定相關人士都沒有動伺服器，只能懷疑被攻擊，畢竟，我貪圖方便直接<strong>在防火牆開Mongodb的27017 port</strong>外加<strong>DB沒有設定auth</strong>…<br>
所以世界上任何人都可以簡單的IP:27017連上MongoDB然後為所欲為。</p>
<p>果然看到Mongod的log記錄:</p>
<pre class=" language-nginx"><code class="prism  language-nginx"><span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.385</span><span class="token operator">+</span><span class="token number">0000</span> I NETWORK  <span class="token punctuation">[</span>thread1<span class="token punctuation">]</span> connection accepted from <span class="token number">207.226</span><span class="token punctuation">.</span><span class="token number">143.128</span><span class="token punctuation">:</span><span class="token number">43072</span>
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.428</span><span class="token operator">+</span><span class="token number">0000</span> I COMMAND  <span class="token punctuation">[</span>conn446<span class="token punctuation">]</span> dropDatabase admin starting
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.475</span><span class="token operator">+</span><span class="token number">0000</span> I COMMAND  <span class="token punctuation">[</span>conn446<span class="token punctuation">]</span> dropDatabase admin finished
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.475</span><span class="token operator">+</span><span class="token number">0000</span> I COMMAND  <span class="token punctuation">[</span>conn446<span class="token punctuation">]</span> setting featureCompatibilityVersion to <span class="token number">3.2</span>
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.505</span><span class="token operator">+</span><span class="token number">0000</span> I <span class="token operator">-</span>        <span class="token punctuation">[</span>conn446<span class="token punctuation">]</span> end connection <span class="token number">207.226</span><span class="token punctuation">.</span><span class="token number">143.128</span><span class="token punctuation">:</span><span class="token function">43072</span> <span class="token punctuation">(</span><span class="token number">6</span> connections now open<span class="token punctuation">)</span>
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.565</span><span class="token operator">+</span><span class="token number">0000</span> I NETWORK  <span class="token punctuation">[</span>thread1<span class="token punctuation">]</span> connection accepted from <span class="token number">207.226</span><span class="token punctuation">.</span><span class="token number">143.128</span><span class="token punctuation">:</span><span class="token number">43076</span>
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.609</span><span class="token operator">+</span><span class="token number">0000</span> I COMMAND  <span class="token punctuation">[</span>conn447<span class="token punctuation">]</span> dropDatabase test starting
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.616</span><span class="token operator">+</span><span class="token number">0000</span> I COMMAND  <span class="token punctuation">[</span>conn447<span class="token punctuation">]</span> dropDatabase test finished
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.647</span><span class="token operator">+</span><span class="token number">0000</span> I <span class="token operator">-</span>        <span class="token punctuation">[</span>conn447<span class="token punctuation">]</span> end connection <span class="token number">207.226</span><span class="token punctuation">.</span><span class="token number">143.128</span><span class="token punctuation">:</span><span class="token function">43076</span> <span class="token punctuation">(</span><span class="token number">6</span> connections now open<span class="token punctuation">)</span>
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.709</span><span class="token operator">+</span><span class="token number">0000</span> I NETWORK  <span class="token punctuation">[</span>thread1<span class="token punctuation">]</span> connection accepted from <span class="token number">207.226</span><span class="token punctuation">.</span><span class="token number">143.128</span><span class="token punctuation">:</span><span class="token number">43078</span>
<span class="token number">2018</span><span class="token operator">-</span><span class="token number">01</span><span class="token operator">-</span>30T03<span class="token punctuation">:</span><span class="token number">59</span><span class="token punctuation">:</span><span class="token number">17.874</span><span class="token operator">+</span><span class="token number">0000</span> I COMMAND  <span class="token punctuation">[</span>conn448<span class="token punctuation">]</span> command Warning<span class="token punctuation">.</span>Readme command<span class="token punctuation">:</span> 
insert <span class="token punctuation">{</span> insert<span class="token punctuation">:</span> <span class="token string">"Readme"</span><span class="token punctuation">,</span> documents<span class="token punctuation">:</span> <span class="token punctuation">[</span> 
	<span class="token punctuation">{</span> 
		_id<span class="token punctuation">:</span> <span class="token function">ObjectId</span><span class="token punctuation">(</span><span class="token string">'5a6fed95b5fdec4f33311aad'</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
		BitCoin<span class="token punctuation">:</span> <span class="token string">"1FwSLsSJPVrWEwX3aQiHr1JqdcQA12M3ro"</span><span class="token punctuation">,</span>
		eMail<span class="token punctuation">:</span> <span class="token string">"mongodb@tfwno.gf"</span><span class="token punctuation">,</span> Exchange<span class="token punctuation">:</span> <span class="token string">"https://localbitcoins.com"</span><span class="token punctuation">,</span>
		Solution<span class="token punctuation">:</span> <span class="token string">"Your Database is downloaded and backed up on our secured servers."</span>
		 <span class="token string">" To recover your lost data: Send 0.1 BTC to our BitCoin Address and Contact us by eMa..."</span> 
	<span class="token punctuation">}</span> 
<span class="token punctuation">]</span>
</code></pre>
<p>直接drop整個資料庫，最後只留下Bitcoin得帳戶所要0.1比特幣，當時查一下匯率…五萬多新台幣…<br>
夠狠! 你行! 🖕<br>
不過還好該網站的資料庫還在測試階段，最近備份也在不久之前，所以簡單作法是直接restore<br>
<code>mongorestore -h 127.0.0.1 -d my-mongo-new --directoryperdb ./mongo-backup/my-mongo</code><br>
但國中導師教我一個觀念</p>
<blockquote>
<p><strong>危機就是轉機</strong></p>
</blockquote>
<p><strong>所以必須重中學到教訓，加強資安措施與增加備援機制</strong></p>
<h2 id="環境">環境</h2>

<table>
<thead>
<tr>
<th>Object</th>
<th>Version</th>
</tr>
</thead>
<tbody>
<tr>
<td>Ubuntu</td>
<td>14.04</td>
</tr>
<tr>
<td>Nodejs</td>
<td>9.3</td>
</tr>
<tr>
<td>MongoDB</td>
<td>3.4.10</td>
</tr>
<tr>
<td>Mongoose</td>
<td>4.13.10</td>
</tr>
</tbody>
</table><h1 id="處理步驟">處理步驟</h1>
<h2 id="step-1-mongodb-authorization">Step 1: MongoDB authorization</h2>
<ol>
<li>預設情況直接開啟Mongod服務會是以下設定 <code>mongod --port 27017 --dbpath /data/db1</code> ，然後連接只需<code>mongo --port 27017</code> or <code>mongo</code> 。</li>
<li>選擇使用admin資料庫並創一個有帳密的使用者</li>
</ol>
<pre class=" language-js"><code class="prism  language-js">use admin
db<span class="token punctuation">.</span><span class="token function">createUser</span><span class="token punctuation">(</span>
  <span class="token punctuation">{</span>
    user<span class="token punctuation">:</span> <span class="token string">"DBowner"</span><span class="token punctuation">,</span>
    pwd<span class="token punctuation">:</span> <span class="token string">"abc123"</span><span class="token punctuation">,</span>
    roles<span class="token punctuation">:</span> <span class="token punctuation">[</span> <span class="token punctuation">{</span> role<span class="token punctuation">:</span> <span class="token string">"DBowner"</span><span class="token punctuation">,</span> db<span class="token punctuation">:</span> <span class="token string">"admin"</span> <span class="token punctuation">}</span> <span class="token punctuation">]</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">)</span>
</code></pre>
<p>其中<code>DBowner</code> 為Super User，<a href="https://docs.mongodb.com/manual/reference/built-in-roles/#superuser-roles">官方文件</a>上說明可以授權(auth)任何user到任何DB，包括自己。<br>
<sub>所以我覺得這會是一個後患，但修復要緊</sub></p>
<ol start="3">
<li>重啟Mongod<br>
<code>mongod --auth</code> or <code>sudo service mongod restart</code></li>
</ol>
<blockquote>
<p>NOTE:<br>
因為後者無法pass --auth參數，所以得多做一個步驟:<br>
<code>sudo vim /etc/mongod.conf</code><br>
找到並修改成</p>
<pre class=" language-git"><code class="prism  language-git"><span class="token deleted">- #security:</span>
<span class="token inserted">+ security:</span>
<span class="token inserted">+    authorization: enabled</span>
</code></pre>
</blockquote>
<ol start="4">
<li>現在開始要操作admin的話都要pass帳密了</li>
</ol>
<pre><code>mongo --port 27017 -u "myUserAdmin" -p "abc123" --authenticationDatabase "admin"
</code></pre>
<p>或是先連上Mongod再進行驗證</p>
<pre class=" language-js"><code class="prism  language-js">$ mongo
<span class="token operator">&gt;</span> use admin
<span class="token operator">&gt;</span> db<span class="token punctuation">.</span><span class="token function">auth</span><span class="token punctuation">(</span><span class="token string">"myUserAdmin"</span><span class="token punctuation">,</span> <span class="token string">"abc123"</span> <span class="token punctuation">)</span>
</code></pre>
<h2 id="step-2-備份回復-restore-database">Step 2: 備份回復 Restore Database</h2>
<p>預設dump出來的資料會放在<code>/data/dump</code> 下，只要到該路徑並使用<code>mongorestore</code></p>
<pre><code>$ cd /data/dump
$ mongorestore -h localhost -d admin -u myUserAdmin -p abc123 --authenticationDatabase admin ./test
</code></pre>
<dl>
<dt>-h</dt>
<dd>host IP</dd>
<dt>-d</dt>
<dd>Database Name</dd>
<dt>-u, -p</dt>
<dd>user name and password</dd>
<dt>--authenticationDatabase</dt>
<dd>該user是在哪個DB create的</dd>
<dt>./test</dt>
<dd>備份的檔案位置(.bson)，當初備份test的資料所以資料夾名稱是test</dd>
</dl>
<h2 id="step-3-更新mongoose連線設定-update-mongoose-connection-setting">Step 3: 更新Mongoose連線設定 Update Mongoose Connection Setting</h2>
<ol>
<li>首先保險起見，給API server一個獨立的user，而且權限縮小，只保留讀寫該DB功能</li>
</ol>
<pre><code>&gt; use admin
&gt; db.createUser({
 user:"node",
 pwd:"node",
 roles:[{role:"readWrite", db:"admin"}]
})
</code></pre>
<ol start="2">
<li>到nodejs設定連接MongoDB的地方</li>
</ol>
<pre class=" language-git"><code class="prism  language-git"><span class="token deleted">- mongoose.connection.openUri('mongodb://localhost/test');</span>
<span class="token inserted">+ mongoose.connection.openUri('mongodb://node:node@localhost/admin');</span>
</code></pre>
<ol start="3">
<li>重啟整個服務</li>
</ol>
<pre><code>$ pm2 stop all
$ pm2 start --interpreter ./node_modules/.bin/babel-node server.js
</code></pre>
<p><img src="https://lh3.googleusercontent.com/RqExzMZB9dQ2YzWaFaYfFNZ7fTfNf0yEP2ypvmTBEEt9Jv-Y5LPt8AT-VBMOSzMrUNL1JXKsIdwpTmkvpBR6f7AluuyyOfl1HL1I-A2MCGaS0Q5gXx-wOX35aX29xLgKVqXG7gPdUrM59Dmq6jo_pjvtA-NIPfhaLT_WcREQE_-qq5BUSHwTaBWFMeQptyvk5DCco4xJ9m2q1ZWh8svnQjOHR7n8obr3qYf6sQZiBxLp_zFH9wNJx1ZAAYW26eE5VhQFkuZ8FMhElgnvLL6ONl5KsHFgWyGhtUdK8qRkA2Q4vC2z1dpm9lP-WbtWqEArNUBHtN1XljtTjptmiyr2rF5qYzwKifW3jHhL6KQPtlShO89EMQYzx_REt8I5lU1EiGW5DBYcwtAs2C3ZbfQw7ZF6BZzWBTPgXlXRFQgLJ-9jtTgcqNDnc1lsogx0kX-EF7Aehh8ipnDj7Nd9p6r5E6CR5YgmBAhC1lEo6wKbBRrYlM5j1ZOkTTVCSZe-UNev7aYtY_QTcn8IyFpHWdBzt6yhluWWXWYiCspMTw2xgUIDG_DJn_lLJ8TP09yqax6e1TMTbAZuzyxme5U8F_DoQL5yoJwgBIhysdEtjlAhv97S_nZ-y3kccfcC0zqc0en1FJBu8_w0EmOV5sK0mIwXf2K38YPmj48r_A=w1620-h957-no" alt="enter image description here"><br>
資料回來了感動QQ</p>
<h2 id="step-4-設定每日備份資料-daily-db-backup">Step 4: 設定每日備份資料 Daily DB Backup</h2>
<p>不管什麼原因，為了防止以後資料再度消失，備份很重要<br>
cron.daily會在每天的凌晨4:02執行該路徑底下的shell script，基本上四點是網站最冷門時段，所以非常符合需求</p>
<ol>
<li><code>sudo vim /etc/cron.daily</code></li>
<li><a href="https://brickyang.github.io/2017/03/02/Linux-%E8%87%AA%E5%8A%A8%E5%A4%87%E4%BB%BD-MongoDB/">借用此教學的內容</a>，依照需求修改即可</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token shebang important">#!/bin/sh</span>
DUMP<span class="token operator">=</span>mongodump
OUT_DIR<span class="token operator">=</span>/data/backup/mongod/tmp   // 备份文件临时目录
TAR_DIR<span class="token operator">=</span>/data/backup/mongod       // 备份文件正式目录
DATE<span class="token operator">=</span><span class="token variable"><span class="token variable">`</span><span class="token function">date</span> +%Y_%m_%d_%H_%M_%S<span class="token variable">`</span></span>    // 备份文件将以备份时间保存
DB_USER<span class="token operator">=</span><span class="token operator">&lt;</span>USER<span class="token operator">&gt;</span>                    // 数据库操作员
DB_PASS<span class="token operator">=</span><span class="token operator">&lt;</span>PASSWORD<span class="token operator">&gt;</span>                // 数据库操作员密码
DAYS<span class="token operator">=</span>14                           // 保留最新14天的备份
TAR_BAK<span class="token operator">=</span><span class="token string">"mongod_bak_<span class="token variable">$DATE</span>.tar.gz"</span> // 备份文件命名格式
<span class="token function">cd</span> <span class="token variable">$OUT_DIR</span>                       // 创建文件夹
<span class="token function">rm</span> -rf <span class="token variable">$OUT_DIR</span>/*                 // 清空临时目录
<span class="token function">mkdir</span> -p <span class="token variable">$OUT_DIR</span>/<span class="token variable">$DATE</span>           // 创建本次备份文件夹
<span class="token variable">$DUMP</span> -u <span class="token variable">$DB_USER</span> -p <span class="token variable">$DB_PASS</span> -o <span class="token variable">$OUT_DIR</span>/<span class="token variable">$DATE</span>  // 执行备份命令
<span class="token function">tar</span> -zcvf <span class="token variable">$TAR_DIR</span>/<span class="token variable">$TAR_BAK</span> <span class="token variable">$OUT_DIR</span>/<span class="token variable">$DATE</span>       // 将备份文件打包放入正式目录
<span class="token function">find</span> <span class="token variable">$TAR_DIR</span>/ -mtime +<span class="token variable">$DAYS</span> -delete             // 删除14天前的旧备份
</code></pre>

