# 🚀 Crypto MACD Sentinel
> **基於 n8n 的全自動加密貨幣 MACD 死叉即時監控與 LINE 警報系統**

[![n8n](https://img.shields.io/badge/Orchestration-n8n-EA4B71?logo=n8n&logoColor=white)](https://n8n.io/)
[![JavaScript](https://img.shields.io/badge/Algorithm-ES6%2B-F7DF1E?logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![LINE](https://img.shields.io/badge/Alert-LINE%20Flex%20Message-00C300?logo=line&logoColor=white)](https://developers.line.biz/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

本專案是一個輕量、低延遲的事件驅動型量化監控工作流。系統定期向台灣 **MAX 交易所** 公開 API 批次取得多幣種 K 線資料，透過手刻的原生 JavaScript 引擎即時計算 EMA 與 MACD 雙軸向量，並在確認收線出現技術面「死叉（Death Cross）」時，自動組裝推播結構化的 **LINE Flex Message** 警報卡片。

---

## 🌟 核心特色

* **自動化排程對齊**：依據 K 線收線週期（預設支援 5 分鐘、30 分鐘等週期）進行定時排程執行。


* **多市場批次並行處理**：單次工作流支援多個交易對（如 `BTC/TWD`、`ETH/TWD`、`DOGE/TWD`、`SHIB/TWD`）的資料擷取與並行分析。


* **零外部套件指標引擎**：使用原生 JavaScript (ES6+) 手刻實作 EMA 與 MACD 演算法，不依賴外部肥大套件，執行效率高且輕量化。


* **防重繪與收線驗證**：精確鎖定剛收線完成的 K 線比對轉折條件（$Hist_{t-1} \ge 0$ 且 $Hist_t < 0$），避免盤中假訊號與重繪（Repaint）問題。


* **視覺化 LINE Flex 推播**：自動組裝包含幣種、收盤價、快慢線數值、柱狀圖差值與精準時間戳的視覺化警報訊息。



---

## 🏗️ 系統架構流程

```text
[ 排程觸發 (Cron) ]        ──>     [ 緩衝等待 (Wait) ]       ──>       [ 市場清單生成 ]
                                                                           │
[ LINE 推播 API ] <── [ Flex 訊息組裝 ] <── [ MACD 指標分析引擎 ] <── [ MAX 公開 REST API ]

```

1. **排程觸發**：依照設定的時間間隔（例如每 5 分鐘或每 30 分鐘）啟動工作流。


2. **緩衝等待**：保留 3～5 秒的緩衝，確保交易所伺服器已完成最新 K 線結算。


3. **市場清單生成**：產生欲監控的幣種代碼陣列，為每個市場建立獨立的運算上下文。


4. **K 線資料擷取**：向 MAX 交易所公開 API（`/api/v2/k`）請求原始 OHLCV 資料。


5. **指標運算引擎**：分割多資產批次資料，依時序計算快線（EMA 12）、慢線（EMA 26）與訊號線（EMA 9），並判斷是否產生死叉。


6. **推播發送**：若符合死叉條件，動態組裝 LINE Flex Message JSON 結構，並透過 Push API 推播至目標用戶。



---

## 🛠️ 技術棧

* **工作流自動化**：n8n (Self-hosted via Docker / Cloud)
* **核心邏輯 / 資料處理**：JavaScript (Node.js ES6+)


* **金融數據來源**：MAX 交易所公開 REST API


* **通訊推播服務**：LINE Messaging API (Flex Message)



---

## 🚀 快速開始與部署說明

### 1. 前置準備

* 一套運行中的 [n8n](https://n8n.io/) 環境。
* 一個 [LINE Developers](https://developers.line.biz/) Messaging API Channel，並取得：
* Channel Access Token (Long-lived)
* Target User ID (您的個人 LINE UID)



### 2. 環境變數設定

複製專案中的 `.env.example` 為 `.env`，並將金鑰填入：

```bash
cp .env.example .env

```

確保填妥下列變數：

* `LINE_CHANNEL_ACCESS_TOKEN`
* `LINE_TARGET_USER_ID`

### 3. 匯入 Workflow

1. 下載專案資料夾中的 `workflows/crypto-macd-alert.json`。
2. 登入 n8n 後台介面。
3. 點選 **Add Workflow** > **Import from File**（或直接將 JSON 貼在畫布上）。


4. 確認最後推播節點的 Token 與目標 ID 設定無誤。


5. 點擊右上角切換為 **Active** 即可開始正式監控。



---

## ⚙️ 參數設定與自訂

| 參數名稱 | 所在節點 | 預設值 | 說明 |
| --- | --- | --- | --- |
| `period` | **抓取MAX K線** | `5`（或 `30`） | K 線週期（單位：分鐘）。

 |
| `limit` | **抓取MAX K線** | `200` | 抓取的歷史 K 棒數量，確保 EMA 數值收斂穩定。

 |
| `FAST` / `SLOW` / `SIG` | **MACD 計算與判定** | `12`, `26`, `9` | 標準 MACD 週期參數。

 |
| `FORCE_TEST_OUTPUT` | **MACD 計算與判定** | `false` | 若設為 `true`，則強制輸出最新訊號以利測試 LINE 推播格式。

 |

---

## ⚠️ 免責聲明

本專案僅供技術研究、教育與個人自動化專案展示用途。工作流所產生的任何指標與通知均不構成任何投資建議或買賣指示，實際交易請自行評估風險。



