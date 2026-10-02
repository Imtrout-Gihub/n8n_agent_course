# 🎭 Playwright CLI 操作教學

> **適用對象：** N8N 第3週課程學生  
> **目標：** 讓 Antigravity Agent 能透過 Playwright CLI 自動操控瀏覽器網頁

---

## 📋 目錄

1. [什麼是 Playwright CLI？](#什麼是-playwright-cli)
2. [前置需求](#前置需求)
3. [Step 1：在 Antigravity 安裝 Playwright CLI](#step-1在-antigravity-安裝-playwright-cli)
4. [Step 2：在 Chrome 安裝 Playwright Extension](#step-2在-chrome-安裝-playwright-extension)
5. [Step 3：設定 MCP Server 讓 Agent 連接 Playwright](#step-3設定-mcp-server-讓-agent-連接-playwright)
6. [Step 4：測試 Agent 操控網頁](#step-4測試-agent-操控網頁)
7. [常見問題排除](#常見問題排除)
8. [實作練習](#實作練習)

---

## 什麼是 Playwright CLI？

**Playwright** 是 Microsoft 開發的瀏覽器自動化工具，可以：

- 🖱️ 模擬點擊、輸入、滾動等滑鼠鍵盤操作
- 📸 截圖、擷取網頁內容
- 🔍 讀取 DOM 元素與頁面資料
- 🚀 自動執行登入、填表、資料擷取等流程

在 **Antigravity** 中，Playwright 以 **MCP（Model Context Protocol）Server** 的形式存在，讓 AI Agent 可以直接下指令操控真實瀏覽器。

```
[Antigravity Agent] ──指令──▶ [Playwright MCP Server] ──操控──▶ [Chrome 瀏覽器]
```

---

## 前置需求

在開始之前，請確認你已安裝：

| 工具 | 版本需求 | 下載連結 |
|------|----------|----------|
| Node.js | v18 以上 | https://nodejs.org |
| Google Chrome | 最新版 | https://www.google.com/chrome |
| Antigravity IDE | 最新版 | 由課程提供 |

確認 Node.js 已正確安裝：

```bash
node -v    # 應顯示 v18.x.x 以上
npm -v     # 應顯示 9.x.x 以上
```

---

## Step 1：在 Antigravity 安裝 Playwright CLI

### 1-1 全域安裝 Playwright MCP Server

打開終端機（Terminal），執行以下指令：

```bash
npm install -g @playwright/mcp
```

安裝完成後，確認安裝成功：

```bash
npx @playwright/mcp --version
```

> ✅ 看到版本號代表安裝成功

### 1-2 安裝 Playwright 瀏覽器核心

```bash
npx playwright install chromium
```

> 💡 **說明：** 這會下載 Playwright 專用的 Chromium 瀏覽器核心（約 100-200MB），與你一般使用的 Chrome 是不同的。

### 1-3 在 Antigravity 設定 MCP Server

在 Antigravity IDE 中，開啟 MCP 設定檔。路徑通常為：

```
~/.gemini/config/mcp_config.json
```

加入以下設定：

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp",
        "--headless"
      ]
    }
  }
}
```

> 💡 `--headless` 表示背景執行（不顯示視窗）。若要看到瀏覽器操作過程，可移除此參數，改為：
> ```json
> "args": ["@playwright/mcp", "--browser", "chrome"]
> ```

### 1-4 重啟 Antigravity IDE

儲存設定後，**完全關閉並重新開啟** Antigravity IDE，讓設定生效。

重啟後，在 Agent 對話框輸入測試指令：

```
請用 Playwright 打開 https://www.bbc.com/business 並截圖
```

Agent 能正確回應代表 MCP Server 已連線成功。

---

## Step 2：在 Chrome 安裝 Playwright Extension

Chrome Extension 讓你可以從瀏覽器介面直接錄製操作、生成 Playwright 腳本，非常適合不熟悉程式碼的同學。

### 2-1 安裝 Extension

1. 點擊以下連結，前往 Chrome Web Store：  
   👉 **[Playwright Extension](https://chromewebstore.google.com/detail/playwright-extension/mmlmfjhmonkocbjadbfplnigmagldckm)**

2. 點擊右上角 **「加到 Chrome」（Add to Chrome）**

   ![安裝按鈕示意](install_button)

3. 出現確認視窗後，點擊 **「新增擴充功能」**

4. 安裝完成後，Chrome 工具列右上角會出現 🎭 圖示

### 2-2 使用 Extension 錄製操作

1. 前往你想要自動化的網頁（例如 `https://www.bbc.com/business`）

2. 點擊工具列的 🎭 **Playwright 圖示**

3. 點擊 **「Start Recording」** 開始錄製

4. 在網頁上進行你想自動化的操作：
   - 點擊按鈕
   - 輸入文字
   - 滾動頁面
   - ...等

5. 操作完成後，點擊 **「Stop Recording」**

6. Extension 會自動產生對應的 **Playwright 程式碼** 或 **CSS Selector**

> 💡 **實用技巧：** 當你不知道某個 HTML 元素的 CSS Selector 是什麼時，用這個 Extension 點擊該元素，就能自動取得！

### 2-3 取得元素的 CSS Selector

這個功能在修改 N8N 的 **HTML Extract 節點**時非常有用：

1. 在目標網頁按下 **F12** 開啟開發人員工具（DevTools）
2. 點擊 Playwright Extension 的 **「Pick Locator」** 按鈕
3. 滑鼠移到你想要的元素上，點擊一下
4. Extension 會顯示最佳的 Playwright Locator（可直接用於 N8N）

---

## Step 3：設定 MCP Server 讓 Agent 連接 Playwright

### 3-1 確認 MCP Server 已啟動

在 Antigravity 對話框，輸入：

```
請列出你現在可以使用的 MCP 工具
```

你應該會看到類似以下的 Playwright 工具列表：

| 工具名稱 | 功能說明 |
|---------|---------|
| `browser_navigate` | 開啟指定網址 |
| `browser_click` | 點擊頁面元素 |
| `browser_type` | 在欄位中輸入文字 |
| `browser_take_screenshot` | 截圖 |
| `browser_snapshot` | 取得頁面 DOM 結構 |
| `browser_find` | 尋找頁面元素 |
| `browser_fill_form` | 自動填寫表單 |
| `browser_wait_for` | 等待元素出現 |

### 3-2 使用 Headed 模式（可以看到瀏覽器）

若要讓瀏覽器操作「可見」，修改 `mcp_config.json`：

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp",
        "--browser", "chrome",
        "--viewport-size", "1280,720"
      ]
    }
  }
}
```

> ⚠️ **注意：** Headed 模式需要圖形介面，如果你在遠端伺服器上執行，請維持 `--headless` 模式。

---

## Step 4：測試 Agent 操控網頁

### 4-1 基本測試：開啟網頁並截圖

在 Antigravity Agent 輸入：

```
請幫我：
1. 打開 https://www.bbc.com/business
2. 等待頁面載入完成
3. 截圖並告訴我頁面標題
```

### 4-2 進階測試：擷取 BBC 新聞標題

```
請用 Playwright 打開 BBC Business 首頁，
找到所有帶有 data-testid="card-headline" 的 h2 元素，
並把所有標題文字列出來給我
```

### 4-3 結合 N8N：讓 Agent 驗證 CSS Selector

當你在 N8N 的 HTML Extract 節點設定 CSS Selector 時，可以請 Agent 先幫你驗證：

```
請打開 https://www.bbc.com/business，
用 CSS Selector `h2[data-testid="card-headline"]` 
找到頁面上的元素，並回傳前 5 個元素的文字內容，
確認這個 Selector 是否有效
```

### 4-4 自動化流程範例：搜尋並截圖

```
請幫我：
1. 打開 https://www.google.com
2. 在搜尋框輸入「BBC Business financial news」
3. 按下搜尋
4. 等待結果頁面載入
5. 截圖保存結果
```

---

## 常見問題排除

### ❌ 問題 1：`npx @playwright/mcp` 找不到指令

**解決方式：**
```bash
# 先確認 npm global 路徑
npm config get prefix

# 重新安裝
npm install -g @playwright/mcp@latest
```

### ❌ 問題 2：Antigravity 找不到 Playwright MCP Server

**解決方式：**
1. 確認 `mcp_config.json` 的格式是合法的 JSON（可用 https://jsonlint.com/ 驗證）
2. 確認已**完全重啟** Antigravity IDE
3. 查看 Antigravity 的 MCP 日誌輸出

### ❌ 問題 3：瀏覽器無法啟動（Linux/遠端環境）

**解決方式：**
```bash
# 安裝必要的系統依賴
npx playwright install-deps chromium
```

### ❌ 問題 4：CSS Selector 抓不到 BBC 元素

**可能原因：** BBC 頁面是 JavaScript 渲染的 SPA，靜態 HTTP 請求只能拿到初始 HTML，動態內容需要等 JS 執行。

**解決方式：** 改用 Playwright 替代 N8N 的 HTTP Request 節點：

```
# 告訴 Agent
請用 Playwright 打開 BBC Business 頁面，
等待 h2[data-testid="card-headline"] 元素出現後，
再擷取所有標題文字
```

### ❌ 問題 5：Chrome Extension 的 Playwright 圖示消失

**解決方式：**
1. 點擊 Chrome 右上角的拼圖 🧩 圖示
2. 找到 Playwright Extension，點擊 📌 釘選

---

## 實作練習

### 🏋️ 練習 A（基礎）
使用 Playwright Extension 錄製以下操作，並觀察產生的 Locator：
1. 打開 `https://www.bbc.com/business`
2. 點擊第一則新聞文章
3. 記下 Playwright Extension 給出的元素 Locator

### 🏋️ 練習 B（中級）
在 Antigravity 請 Agent 完成以下任務：
1. 用 Playwright 打開 BBC Business
2. 擷取前 5 則新聞的標題 + 時間戳
3. 將結果整理成 Markdown 表格

### 🏋️ 練習 C（進階）
修改本週的 N8N Market News 工作流程：
1. 將 **HTTP Request 節點**改為透過 Playwright 抓取（讓 JavaScript 渲染完再讀取）
2. 讓 Agent 驗證抓到的標題數量比純 HTTP 方式更多
3. 比較兩種方式的差異，寫下你的觀察

---

## 📚 延伸資源

- [Playwright 官方文件](https://playwright.dev/docs/intro)
- [Playwright MCP GitHub](https://github.com/microsoft/playwright-mcp)
- [CSS Selector 速查表](https://www.w3schools.com/cssref/css_selectors.php)
- [Antigravity MCP 設定指南](../.agents/)

---

> 💬 **有問題嗎？** 在課堂 Discord 的 `#week3-playwright` 頻道發問，助教會協助解答！

---

*最後更新：2026-10-02 | 第3週 Outlook 與 API 整合課程*
