# uv：終結 Python 套件管理地獄

## 為什麼寫這篇

最近接案要裝 LangChain，結果被依賴衝突卡了一整個下午。pydantic v2 和 v1 的版本打架，本機能跑的東西到客戶那邊就炸掉，明明有 `requirements.txt` 但每次重裝環境都會遇到新問題。

就在快放棄的時候，看到「Youtube」的影片推給我 [uv](https://www.youtube.com/watch?v=jd1aRE5pJWc)。看完才發現，原來我一直在用石器時代的方法管理Python專案。

---

## 傳統方法的痛點

### 環境污染

剛學 Python 時都是直接 `pip install`，結果所有套件裝到全域。等你做第二個專案，需要不同版本的套件時就炸了。專案 A 要 Flask 3.0，專案 B 要 Flask 2.3，你只能裝一個。

雖然有虛擬環境可以解決，但你要記得每次都啟動。忘記啟動就直接裝套件？又污染全域了。

### 依賴地獄

裝一個 Flask，實際上會連帶裝 Werkzeug、Jinja2、click 等六個套件。這些是「間接依賴」——你不直接用，但 Flask 需要它們。

問題是當你 `pip uninstall flask`，只會移除 Flask 本身，其他六個套件留在那邊變成垃圾。做幾個專案後，環境裡堆滿了這些孤兒依賴，你根本不知道哪些可以刪。

### requirements.txt 的問題

`pip freeze` 會把環境中所有套件都列出來，你分不清哪些是直接需要的、哪些是間接依賴。而且它只記錄版本範圍，不記錄確切版本。

結果就是：同樣的 `requirements.txt`，今天裝和三個月後裝，環境可能不一樣。因為某些套件發布了新版本，依賴關係也跟著變了。

這就是為什麼常常聽到「在我電腦上可以跑」這種話。

尤其是沒有指定版本的狀況，更慘。

```bash
fastapi
python-dotenv
pandas
langchain
```

在requirements.txt中，像這樣沒有寫套件名稱，就是固定抓最新版本。之後必有衝突。

### 工具太分散

管理 Python 版本要 pyenv，建虛擬環境要 venv，裝套件要 pip，鎖版本要 pip-tools，打包要 setuptools。每個工具有自己的指令、自己的設定檔，學習成本很高。

而且 pip 很慢。每次都要重新下載、重新編譯，就算昨天才裝過，今天重建環境還是要再等一次。

---

## uv 如何解決這些問題

uv 是 Astral 團隊（做 Ruff 的那個團隊）開發的工具，目標就是把所有東西整合成一個。

### 核心功能

**統一所有工具**

不用再裝一堆工具了。Python 版本管理、虛擬環境、套件安裝、版本鎖定，全部用 uv 搞定。

```bash
# 安裝 Python 版本
uv python install 3.12

# 初始化專案
uv init my-project

# 安裝套件
uv add flask

# 執行程式（自動在虛擬環境執行）
uv run python app.py
```

而且很讚的是，虛擬環境是自動管理的，你甚至感覺不到它的存在。不用手動建立、不用手動啟動。

**依賴分離**

uv 用 `pyproject.toml` 記錄你直接需要的套件，用 `uv.lock` 記錄完整的依賴樹（包含所有間接依賴的確切版本）。

這樣協作的時候，其他人一眼就知道專案的核心依賴是什麼。而且 `uv.lock` 確保不管在哪個環境、什麼時候安裝，版本都完全一樣。

```bash
# 其他人拿到你的專案
git clone your-project
uv sync  # 一鍵復現完全相同的環境
```

**自動清理**

移除套件的時候，uv 會自動分析依賴關係，把那些沒人用的間接依賴一起清掉。不會留下孤兒依賴。

**極致的速度**

uv 用 Rust 寫的(Rust真是太棒了😍)，加上全域快取和平行下載，比 pip 快非常多。實測安裝 langchain，pip 要好幾個小時，uv 只要 5~10 分鐘。而且第二次安裝（有快取）只要 2-3 秒。

---

## 指令對照表
這裡我整理了一些跟pip對照的指令，來讓大家好上手！

| 功能 | 傳統方法 (venv + pip) | uv 指令 |
|------|---------------------|---------|
| **安裝 Python** | 到官網下載安裝檔 | `uv python install 3.12` |
| **初始化專案** | 手動建立目錄和檔案 | `uv init` |
| **建立虛擬環境** | `python -m venv .venv` | 自動處理，無需手動建立 |
| **啟動虛擬環境** | `source .venv/bin/activate` (Mac/Linux)<br>`.\venv\Scripts\activate` (Windows) | 無需啟動 |
| **安裝套件** | `pip install flask` | `uv add flask` |
| **安裝開發工具** | `pip install pytest` | `uv add --dev pytest` |
| **移除套件** | `pip uninstall flask` | `uv remove flask` |
| **鎖定版本** | `pip freeze > requirements.txt` | 自動產生 `uv.lock` |
| **復現環境** | `pip install -r requirements.txt` | `uv sync` |
| **執行程式** | `python app.py` | `uv run python app.py` |
| **執行測試** | `pytest` | `uv run pytest` |
| **查看依賴樹** | 需要安裝 `pipdeptree` | `uv tree` |
| **切換 Python 版本** | 需要安裝 `pyenv` | `uv python pin 3.12` |
| **安裝全域工具** | 需要安裝 `pipx` | `uv tool install ruff` |

### 實際應用

**建立新專案**
```bash
# 傳統方法
mkdir my-project && cd my-project
python -m venv .venv
source .venv/bin/activate
pip install flask
pip freeze > requirements.txt

# uv
uv init my-project
cd my-project
uv add flask
```

**協作復現環境**
```bash
# 傳統方法
git clone project
cd project
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# uv
git clone project
cd project
uv sync
```

**執行程式**
```bash
# 傳統方法
source .venv/bin/activate
python app.py

# uv
uv run python app.py
```

---

## 使用心得

這次用過uv之後才讓我真正意識到：Python 開發根本就不該這麼折磨人。明明是很好上手的語言，環境配置卻很折磨。
傳統的 requirements.txt + pip 雖然是標準，但實際上卻讓環境充滿難以追蹤的依賴問題，「本機能跑、部署爆炸」早已是常態。我之前也為了裝一個套件裝好幾個小時。找依賴、換python版本...

直到現在接觸 uv，才發現原來 Python 的版本、虛擬環境與套件管理是可以被一次整合、而且做到又快又乾淨的。根本不需要管那些雜七雜八的東西。
需要什麼套件直接說，給uv處理就好了。

如果你曾被複雜套件或環境問題卡到心累，uv 真的值得一試。
用過之後，只能說相見恨晚。

---

## 參考資料

- [uv 官方文件](https://docs.astral.sh/uv/)
- [GitHub - astral-sh/uv](https://github.com/astral-sh/uv)
- [程序员老王 - uv 教學影片](https://www.youtube.com/watch?v=jd1aRE5pJWc)