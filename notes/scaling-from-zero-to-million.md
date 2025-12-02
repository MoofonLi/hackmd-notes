# SCALE FROM ZERO to MILLION of USERS

## 何謂系統？

維基百科對「系統」（System）的定義是：
「一群有關聯的個體，根據某種規則運作，能完成單一元件無法獨立達成的任務。」簡單說，系統就是「讓許多部分協作」的結構。

做一個只有自己在用的系統很簡單。
但當使用者數量上升，一切都變了。

想想你每天滑的 Threads —— 你看到的內容可能和別人不一樣，有時還會延遲。
或者像 AWS 當機時，Canva、Duolingo 全部無法使用。

這些都是「系統設計」要解決的問題：
**當使用者從 1 個變成 100 萬個時，系統要如何保持穩定、快速又一致？**

---

## Single Server Setup（單一伺服器架構）

再怎麼大的系統，一開始都只是在一台機器上運行。這也是系統設計的起點。

想像一位使用者要訪問你的網站，流程大概是這樣：

![upload_3d5646cee4b84ef9398839ff6dae94a3](https://hackmd.io/_uploads/B1bvxvgxWe.png)

1. 使用者在瀏覽器輸入 `www.example.com`
2. 瀏覽器或手機 App 先去查 **DNS**（Domain Name System）
3. DNS 把網址轉成 **IP 位址**（像是 `93.184.216.34`）
4. 拿到 IP 後，發送 **HTTP 請求**到 Web Server
5. Web Server 去資料庫撈資料，回傳給使用者

**這個流程中出現的角色：**
- **USER**：就是你我，用電腦或手機發請求的人。我在底下有補充[電腦跟手機訪問網站的差別](#1-Web-Application-vs-Mobile-Application)，有興趣可以看
- **IP**：伺服器的地址，就像門牌號碼
- **DB**：存資料的地方，帳號、文章、商品資料都在這
- **DNS**：[底下有補充](#2-DNS%EF%BC%88Domain-Name-System%EF%BC%89)

---

## Database（資料庫）

使用者越來越多，一台伺服器快撐不住了。怎麼辦？

最直接的做法就是把「處理請求」和「存資料」分開。

把 Web/Mobile traffic（web tier）和 Database（data tier）分開後，它們可以各自擴展。流量爆了？加 web server。資料太多？升級資料庫。互不干擾。

### 該選哪種資料庫？

要先決定用 **Relational Database** 還是 **Non-Relational Database**。

可以參考我這篇 [Relational Database vs Non-Relational Database](https://hackmd.io/@Moofon/HyfVei9xWe)

**Relational Database（RDBMS / SQL Database）**

MySQL、PostgreSQL、Oracle 這些。用 table 跟 row 存資料，可以用 SQL 做 join。發展超過 40 年了，很穩。

**Non-Relational Database（NoSQL Database）**

像 MongoDB、Redis、Cassandra、DynamoDB。分成 key-value stores、graph stores、column stores、document stores 四大類。通常不支援 join。

其實對大部分人來說，**關聯式資料庫夠用了**。但如果符合以下情況，可以考慮 NoSQL：

- 需要超低延遲
- 資料沒什麼關聯
- 只是要做序列化跟反序列化（JSON、XML 之類）
- 要存超大量資料

像我最近做的專案，在網路上爬了大量文本來餵給 AI 做 RAG，幾百萬個字。我就選 NoSQL，因為這些文本資料之間沒什麼關聯。如果用關聯式資料庫，應該會有問題。

---

## Vertical Scaling vs Horizontal Scaling

資料量每天都在增長，資料庫遲早撐不住。這時候要擴展。而擴展又分成Vertical Scaling(垂直擴展) 與 Horizontal Scaling(水平擴展)。

![1_gee5Zkih2dZ7tYWRgmRbkw](https://hackmd.io/_uploads/r1aLsoKgWl.png)


### Vertical Scaling（Scale Up）

**簡單說就是換台更猛的機器。**

加 CPU、加 RAM、加硬碟。根據 Amazon RDS，你可以買到有 24 TB RAM 的資料庫伺服器。這種怪獸機器確實能處理大量資料。

像 2013 年的 Stack Overflow 每月有 1000 多萬訪客，但只用一台 master database。

但 vertical scaling 有幾個問題：

- **硬體有極限**。不可能無限加 CPU 跟 RAM
- **單點故障**。這台掛了，整個系統就掛了
- **超貴**。越強的機器越貴，而且不是線性成長

### Horizontal Scaling（Scale Out / Sharding）

**不是換更強的機器，是加更多機器。**

![DB_image_3_cropped](https://hackmd.io/_uploads/BypwhsYgZx.png)


Sharding 就是把大型資料庫切成更小的 shards。每個 shard schema 相同，但存的資料各自獨立。

假設我們用 `price 價格區間` 來分配：

```
0 < price < 49.99 → Shard 0
50 < price < 99.99 → Shard 1
price > 100 → Shard 2
```

每次存取資料時，用 hash function 算出要去哪個 shard。

**Sharding Key 的選擇很重要**。選不好的話，某些 shard 會過載。

流量小的時候，vertical scaling 簡單有效。但對大規模應用來說，**horizontal scaling 是唯一出路**。

現在幾乎都是用水平擴展，除非是小專案。不過 [Sharding](#3-Sharding-%E7%9A%84%E5%95%8F%E9%A1%8C) 也會帶來新問題，最底下有說明。

---

## Load Balancer（負載平衡器）

單一 web server 有兩個致命問題：

1. **Single Point of Failure**：Server 掛了，網站就掛了
2. **無法處理高流量**：流量暴增時，單台 server 會被打爆

Load Balancer 的作用就是把請求平均分配給多台 web servers。

![54114fcc-cce5-4f0d-a678-47f8f0339fd4_1304x1076](https://hackmd.io/_uploads/SySuh2sbWg.jpg)


> 💡 **圖片說明**：使用者(Public IP) → Load Balancer → 分配請求(Private IP)給多台 Web Servers

現在，使用者連到 Load Balancer 的 [Public IP](#4-Public-IP-vs-Private-IP)，而非直接連 web server。Load Balancer 再決定要把請求送給哪台 server。

### 解決單點故障

假設現在有 Server 1 和 Server 2。

- Server 1 掛了 → Load Balancer 自動把所有流量導到 Server 2
- 同時啟動新的 server 補上

使用者完全無感，網站不會中斷。

### 自動擴展

流量爆增時，只要加更多 web servers，Load Balancer 會自動開始分配流量給新 servers。

流量下降時，關掉幾台 server 省成本。這就是 horizontal scaling 的優勢。

之前在實習的時候，半夜 server 突然掛掉，還好有 Load Balancer 自動 failover 到備用 server。不然隔天就準備出事了xd。

---

## Database Replication（資料庫複製）

單一 database 的問題跟單一 web server 一樣：掛了就全毀。

Database Replication 透過 **Master-Slave 架構**來解決。

![Database Replication 架構圖](https://hackmd.io/_uploads/HkZqo6ibWg.png)


> 💡 **圖片說明**：Master DB (寫入) → 自動複製 → 多台 Slave DB (讀取)

- **Master Database**：只負責寫入（insert, delete, update）
- **Slave Database**：只負責讀取（read）

Master 的資料會自動同步到 Slaves。大部分應用讀取遠多於寫入，所以通常會有多台 Slaves。

### 效能提升

寫入集中在 Master，讀取分散到多台 Slaves 並行處理。

由於讀取遠大於寫入。像是大家在電商平台更多時間都是再逛逛，沒有實際在買東西。所以多了專門用來讀取的slave db後，查詢速度會明顯變快很多。

### 可靠性與高可用性

資料被複製到多個地方。某台 database server 硬碟壞掉、被天災毀掉，至少資料還有備份。

而且任一台 database 掛了，系統還能繼續運作，至於要如何處理掛掉的db，在底下有[補充說明](#5-DB-Failover-%E8%99%95%E7%90%86)。

---

### 現在的架構

![完整架構](https://hackmd.io/_uploads/Byiw6Ts-Zl.png)

> 💡 **圖片說明**：DNS → Load Balancer → Web Servers → Master/Slave DBs

完整流程：
1. 使用者從 DNS 拿到 Load Balancer 的 IP
2. Load Balancer 把請求分配給某台 Web Server
3. Web Server 從 Slave Database 讀取資料
4. 寫入、修改、刪除操作會送到 Master Database
5. Master 自動同步資料到 Slaves

### 還不夠快

這一篇我們完成了：

* ✅ 系統與資料庫概覽
* ✅ Load Balancer 分散流量
* ✅ Database Replication 避免單點故障
* ✅ Private IP 保護內部 servers

**雖然系統穩定了，但載入速度還不夠快！下一篇會聊如何用 Cache 和 CDN 來優化載入速度。**

---

## 補充說明

### 1. Web Application vs Mobile Application

伺服器的流量主要來自兩邊：**Web Application** 跟 **Mobile Application**。

**Web Application**

Web App 通常後端用 Java、Python 處理邏輯跟資料，前端用 HTML、CSS、JavaScript 顯示內容。

**Mobile Application**

Mobile App 透過 **HTTP 協定**跟伺服器溝通，常用 **JSON 格式**傳資料，因為輕量又好讀。

舉例，當 App 發請求：
```
GET /users/12
```

伺服器回傳使用者資料：
```json
{
  "id": 12,
  "name": "Moofon",
  "role": "Goblin"
}
```

這就是最基本的 API 請求。

---

### 2. DNS（Domain Name System）

DNS 就像網路世界的電話簿，把 `www.example.com` 這種人類看得懂的網址，轉成機器看得懂的 IP 位址（像 `93.184.216.34`）。

流程：
- 使用者輸入網址
- DNS 查對應的 IP
- 回傳給瀏覽器

💡 就像你要去一家餐廳（www.example.com），DNS 就是 Google Maps 告訴你地址在哪（93.184.216.34）。

---

### 3. Sharding 的問題

Sharding 很強，但也有麻煩的地方：

**Resharding data（重新分片）**

當某個 shard 資料增長太快，要重新分配資料、改 sharding function。這過程很煩，系統可能還要暫時停機。

解決方法是用 Consistent Hashing（一致性雜湊），後面章節會講。

**Celebrity problem（Hotspot key problem）**

名人問題。想想我的 IG 粉絲跟 NBA 球星差多少？如果 Instagram 用 Sharding，把 LeBron、Curry、KD 都放同一個 shard，那個 shard 會因為請求太多爆掉。

解決方法是給熱門資料獨立的 shard。

**Join 和 De-normalization**

資料庫被 shard 後，跨 shard 的 join 會超慢。

比如 User 在 Shard 3，Order 散在其他 shards，要組合起來很麻煩。

**解決方法：De-normalization**

就是把資料重複存。Order 表格不只存 user_id，連 user 的名字、email 都直接存進去。查詢時就不用跨 shard join 了。

缺點是資料重複佔空間，更新時要改多個地方。但這就是 sharding 的取捨：**用空間換速度**。

---

### 4. Public IP vs Private IP

是電資出身的人一定對這個不陌生，IP 是網路最重要的概念。而IP又分成～

- **Public IP**：Load Balancer 對外的 IP，使用者連這個
- **Private IP**：Web servers 之間溝通用的內部 IP

所以 Load Balancer 的設計有個重要細節：

Web servers 只有 Private IP，外網連不到。駭客只能攻擊 Load Balancer 這層，打不進後面的 servers。

其實與你家裡的 Wifi 路由器的概念很類似，**不過是反過來的！**：

![Private-vs-Public_2-1024x455](https://hackmd.io/_uploads/Hyce1aiZ-e.png)

路由器有對外的 Public IP（中華電信給的），而手機、電腦都是內部的 Private IP（192.168.x.x）。

當你上網時：電腦透過內部 IP 連到路由器 → 路由器用 Public IP 對外溝通 → 外面的人只看得到路由器，看不到你家裡的裝置。

---

### 5. DB Failover 處理

**Slave 掛了**

流量會自動導到其他健康的 Slaves，同時啟動新的 Slave 來替換。

**Master 掛了**

某台 Slave 會被提升為新的 Master，接手寫入工作。但這比較複雜，因為 Slave 的資料可能還沒完全同步，需要跑 data recovery scripts 來補齊。

---

## 參考資料
1. https://medium.com/@arumugamperumal471/scaling-from-one-to-million-single-server-setup-33998e5adadc
2. System Design Interview – An Insider's Guide by Alex Xu
3. https://miro.medium.com/1*gee5Zkih2dZ7tYWRgmRbkw.png
4. https://assets.digitalocean.com/articles/understanding_sharding/DB_image_3_cropped.png
5. https://substackcdn.com/image/fetch/$s_!jpjg!,w_1200,h_600,c_fill,f_jpg,q_auto:good,fl_progressive:steep,g_auto/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F54114fcc-cce5-4f0d-a678-47f8f0339fd4_1304x1076.png