# SCALE FROM ZERO TO MILLION of USERS

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

千里之行，始於第一台伺服器。
所有的偉大系統，一開始都只是在一台機器上運行。
這是最簡單、最純粹的架構，也是系統設計的起點。

想像一位使用者要訪問你的網站，我們來看一下這個簡單的流程：

![upload_3d5646cee4b84ef9398839ff6dae94a3](https://hackmd.io/_uploads/B1bvxvgxWe.png)

當使用者透過域名（例如 `www.example.com`）訪問你的網站時，整個請求流程是這樣的：

1. 首先，使用者的瀏覽器或手機 App 會先查詢 **DNS**（Domain Name System，域名系統）
2. DNS 會把域名轉換成 **IP 位址**（例如 `93.184.216.34`）
3. 當瀏覽器拿到 IP 位址後，就會發送 **HTTP 請求**到你的 Web Server
4. Web Server 接收到請求後，會查詢資料庫取得資料，然後將資料回傳給使用者

**當中的重要角色：**
- **USER**：發出請求的人或裝置，例如使用者的電腦或手機
- **IP**：伺服器在網路上的地址，讓資料知道該送去哪
- **DB**：存放應用程式資料的地方，例如帳號、商品或文章內容
- **DNS**：以下補充

---
## 補充說明

### 1. DNS（Domain Name System）

DNS 是整個網路世界的「電話簿」。
它的任務是把人類好記的網址（例如 `www.example.com`）
轉換成機器能理解的 IP 位址（例如 `93.184.216.34`）。

流程像這樣：
- 使用者輸入網址
- DNS 查找對應的 IP
- 回傳結果給瀏覽器

💡 可以想像你要去一家餐廳（www.example.com），
DNS 就像是 Google Maps 告訴你「它的地址在 XX 路 XX 號」(93.184.216.34)。

### 2. Web Application vs Mobile Application

伺服器的流量主要來自兩個方向：
**Web Application** 和 **Mobile Application**。

**Web Application**

Web App 通常由後端（如 Java、Python）處理邏輯與資料存取，前端使用 HTML、CSS、JavaScript 呈現內容。
這種架構靈活且容易維護，是大多數網站的基礎。

**Mobile Application**

Mobile App 則透過 **HTTP 協定**與伺服器交換資料，
常使用 **JSON 格式**傳輸，因為它輕量、易讀。

舉例來說，當 App 發出請求：
```
GET /users/12
```

伺服器就會回傳該使用者（ID = 12）的資料，
例如：
```json
{
  "id": 12,
  "name": "Moofon",
  "role": "Goblin"
}
```

這就是一個最基本的 API 請求。


## 參考資料: 
1. https://medium.com/@arumugamperumal471/scaling-from-one-to-million-single-server-setup-33998e5adadc
2. System Design Interview – An insider's guide