# PageAgent Offline Runner

在任何網頁的 Chrome DevTools 中執行 AI 瀏覽器自動化，**無須安裝 Chrome Extension、無須網路連線**。

基於 [PageAgent](https://github.com/alibaba/page-agent)（阿里巴巴開源，page-agent@1.6.2）。

---

## 與原版的差異

本專案對原始碼進行以下修改：

| # | 項目 | 說明 |
|---|------|------|
| 1 | **移除預設中國 AI** | 原版預設連接阿里雲 Qwen 示範端點；本版改為由使用者自行輸入，不預設任何 AI 服務 |
| 2 | **離線核心，無須 CDN** | 原版透過 CDN（jsdelivr / npmmirror）動態載入核心程式；本版已將核心程式內嵌，執行時完全不需要對外連網拉取腳本 |
| 3 | **外連 API，需自備 API Key** | AI 推理仍需呼叫你指定的 API 端點（雲端模型需對外網路連線），請自備對應服務的 API Key |
| 4 | **支援本地 AI 模型（隱私保護）** | 可接 Ollama 等本地模型，所有資料留在本機不外傳；⚠️ 目前本地模式因 **CORS 限制尚未完全解決**，部分網站可能無法正常呼叫 |

---

## 著作權

Copyright © 2025 乙仔. All rights reserved.

本專案採用**非商業授權**：

- ✅ **允許**：個人使用、學術研究、非營利開源專案
- ❌ **禁止**：任何形式的商業使用、內嵌於商業產品或服務

如需商業授權，請聯繫作者取得授權。

### 上游來源

本專案內嵌並二次開發自以下開源專案：

| 項目 | 說明 |
|------|------|
| **專案名稱** | [alibaba/page-agent](https://github.com/alibaba/page-agent) |
| **版本** | v1.6.2 |
| **原始授權** | MIT License |
| **原始作者** | Alibaba / PageAgent.js Team |

原始專案依其 MIT License 條款授權使用；本專案之二次開發部分（離線啟動器、設定儲存邏輯）著作權歸乙仔所有，並採上述非商業授權條款。

---

## 原理

PageAgent 是一個「住在網頁裡」的 GUI Agent，運作流程如下：

```
使用者輸入指令
      ↓
PageAgent 序列化當前 DOM（快照頁面結構）
      ↓
傳送給 LLM（你的 AI 模型）
      ↓
LLM 回傳動作指令（點擊哪個元素、輸入什麼文字）
      ↓
PageAgent 用瀏覽器原生 DOM API 執行動作
      ↓
截取新 DOM 快照，重複循環
```

整個流程完全在頁面內的 JavaScript 執行，**apiKey 只會送到你自己設定的 baseURL**，不會洩漏到第三方。

---

## 限制

| 限制 | 原因 |
|------|------|
| **不支援跨頁導航** | 點擊連結跳轉新頁面後，當前頁面 JS runtime 被摧毀，PageAgent 實例隨之消失 |
| **不支援新分頁** | 同上，新分頁是獨立的 JS context |
| 需要支援 Function Calling 的模型 | PageAgent 使用 Tool Use 傳遞操作指令 |

> **解決跨頁需求**：需使用 Chrome Extension 或 MCP Server（PageAgent 官方 MCP Server 也需要 Chrome Extension 作為橋接）。

---

## 使用方法

### 方法一：DevTools Snippets（推薦，可永久保存）

1. 開啟任意網頁，按 **F12** 開啟 DevTools
2. 切換到 **Sources(來源)** 分頁 → 左側展開 **Snippets(程式碼片段)**
3. 點 **+ New snippet(+新的程式碼片段)**，命名為 `pageagent-offline`
4. 將 `page-agent-offline.js` 的全部內容貼入
5. 按 **Ctrl+Enter** 執行

### 方法二：本機伺服器（適合多裝置使用）

```powershell
cd D:\Python\PageAgent
python -m http.server 8080
```

然後在任何網頁的 DevTools Console 貼入：

```javascript
var s = document.createElement("script");
s.src = "http://localhost:8080/page-agent-offline.js";
document.body.appendChild(s);
```

---

## 首次執行設定

執行後會依序彈出 3 個輸入框：

| 步驟 | 欄位 | 說明 |
|------|------|------|
| 1/3 | **baseURL** | AI API 的端點網址 |
| 2/3 | **apiKey** | API 金鑰 |
| 3/3 | **model** | 模型名稱 |

**設定會自動儲存到 `localStorage`**，下次執行時輸入框會預填上次的值，直接按 Enter 即可。

---

## AI 模型設定方法

### Google Gemini（Google AI Studio）

```
baseURL:  https://generativelanguage.googleapis.com/v1beta/
apiKey:   （從 Google AI Studio 取得）
model:    gemini-flash-latest
```

前往 [Google AI Studio](https://aistudio.google.com) 申請免費 API Key。

> **注意**：請選用支援 Function Calling 的模型（標記 ✅ 者），純音訊、圖片生成、嵌入向量等模型不支援瀏覽器操作自動化。

#### 支援 Function Calling 的推薦模型

| 模型名稱 | 最大輸入 token | 最大輸出 token |
|---------|:-------------:|:-------------:|
| `gemini-flash-latest` ⭐ | 1,048,576 | 65,536 |
| `gemini-pro-latest` | 1,048,576 | 65,536 |
| `gemini-flash-lite-latest` | 1,048,576 | 65,536 |
| `gemini-2.5-flash` | 1,048,576 | 65,536 |
| `gemini-2.5-pro` | 1,048,576 | 65,536 |
| `gemini-2.5-flash-lite` | 1,048,576 | 65,536 |
| `gemini-3-flash-preview` | 1,048,576 | 65,536 |
| `gemini-3-pro-preview` | 1,048,576 | 65,536 |
| `gemini-3.1-flash-lite-preview` | 1,048,576 | 65,536 |
| `gemini-3.1-pro-preview` | 1,048,576 | 65,536 |
| `gemini-2.0-flash` | 1,048,576 | 8,192 |
| `gemini-2.0-flash-lite` | 1,048,576 | 8,192 |
| `gemma-3-27b-it` | 131,072 | 8,192 |
| `gemma-3-12b-it` | 32,768 | 8,192 |
| `gemma-3-4b-it` | 32,768 | 8,192 |

<details>
<summary>📋 查看完整模型清單（含不適用模型）</summary>

完整清單來源：[ray.so/m6j9Wam](https://ray.so/m6j9Wam)

| 模型名稱 | 輸入 token | 輸出 token |
|---------|----------:|----------:|
| gemini-2.5-flash | 1,048,576 | 65,536 |
| gemini-2.5-flash-lite | 1,048,576 | 65,536 |
| gemini-2.5-flash-lite-preview-09-2025 | 1,048,576 | 65,536 |
| gemini-2.5-pro | 1,048,576 | 65,536 |
| gemini-3-flash-preview | 1,048,576 | 65,536 |
| gemini-3-pro-preview | 1,048,576 | 65,536 |
| gemini-3.1-flash-lite-preview | 1,048,576 | 65,536 |
| gemini-3.1-pro-preview | 1,048,576 | 65,536 |
| gemini-3.1-pro-preview-customtools | 1,048,576 | 65,536 |
| gemini-flash-latest | 1,048,576 | 65,536 |
| gemini-flash-lite-latest | 1,048,576 | 65,536 |
| gemini-pro-latest | 1,048,576 | 65,536 |
| gemini-robotics-er-1.5-preview | 1,048,576 | 65,536 |
| lyria-3-clip-preview | 1,048,576 | 65,536 |
| lyria-3-pro-preview | 1,048,576 | 65,536 |
| gemini-2.0-flash | 1,048,576 | 8,192 |
| gemini-2.0-flash-001 | 1,048,576 | 8,192 |
| gemini-2.0-flash-lite | 1,048,576 | 8,192 |
| gemini-2.0-flash-lite-001 | 1,048,576 | 8,192 |
| deep-research-pro-preview-12-2025 | 131,072 | 65,536 |
| gemini-2.5-computer-use-preview-10-2025 | 131,072 | 65,536 |
| gemini-3.1-flash-live-preview | 131,072 | 65,536 |
| gemini-3-pro-image-preview | 131,072 | 32,768 |
| nano-banana-pro-preview | 131,072 | 32,768 |
| gemini-2.5-flash-native-audio-latest | 131,072 | 8,192 |
| gemini-2.5-flash-native-audio-preview-09-2025 | 131,072 | 8,192 |
| gemini-2.5-flash-native-audio-preview-12-2025 | 131,072 | 8,192 |
| gemma-3-27b-it | 131,072 | 8,192 |
| gemini-3.1-flash-image-preview | 65,536 | 65,536 |
| gemini-2.5-flash-image | 32,768 | 32,768 |
| gemma-3-12b-it | 32,768 | 8,192 |
| gemma-3-1b-it | 32,768 | 8,192 |
| gemma-3-4b-it | 32,768 | 8,192 |
| gemini-2.5-flash-preview-tts | 8,192 | 16,384 |
| gemini-2.5-pro-preview-tts | 8,192 | 16,384 |
| gemma-3n-e2b-it | 8,192 | 2,048 |
| gemma-3n-e4b-it | 8,192 | 2,048 |
| gemini-embedding-2-preview | 8,192 | 1 |
| aqa | 7,168 | 1,024 |
| gemini-embedding-001 | 2,048 | 1 |
| imagen-4.0-fast-generate-001 | 480 | 8,192 |
| imagen-4.0-generate-001 | 480 | 8,192 |
| imagen-4.0-ultra-generate-001 | 480 | 8,192 |
| veo-2.0-generate-001 | 480 | 8,192 |
| veo-3.0-fast-generate-001 | 480 | 8,192 |
| veo-3.0-generate-001 | 480 | 8,192 |
| veo-3.1-fast-generate-preview | 480 | 8,192 |
| veo-3.1-generate-preview | 480 | 8,192 |

</details>

### 本機 Ollama

```
baseURL:  http://localhost:11434/v1
apiKey:   ollama
model:    qwen2.5
```

> **注意**：模型必須支援 **Function Calling / Tool Use**，否則 PageAgent 無法正常運作。

#### Ollama CORS 設定（必要）

瀏覽器呼叫本機 Ollama 需先開放跨來源：

```powershell
# 單次啟動
$env:OLLAMA_ORIGINS = "*"
ollama serve
```

```powershell
# 永久設定
[System.Environment]::SetEnvironmentVariable("OLLAMA_ORIGINS", "*", "User")
```

驗證連線：

```javascript
fetch("http://localhost:11434/v1/models")
  .then(r => r.json())
  .then(d => console.log(d))
```

---

## 常用指令

### 重新執行（不需重新輸入設定）

在 DevTools Snippets 按 **Ctrl+Enter**，輸入框預填上次設定，直接按 Enter 三次即可。

### 清除儲存的設定

```javascript
localStorage.removeItem("pageAgentConfig")
```

### 手動停止 PageAgent

```javascript
window.pageAgent.dispose()
```

### 查看目前設定

```javascript
console.log(window.pageAgent.config)
```

---

## 檔案說明

| 檔案 | 說明 |
|------|------|
| `page-agent-offline.js` | 完整離線腳本（含 PageAgent 1.6.2 函式庫 + 設定啟動器） |
| `README.md` | 本文件 |

---

## 關於 apiKey 安全性

經原始碼分析確認：

- apiKey **只會**傳送到你自己設定的 `baseURL`
- 原始碼內**無任何遙測或第三方呼叫**
- 整個 PageAgent 在本地瀏覽器 JS 環境內執行

---

## 參考資料

- [PageAgent 官方文件](https://alibaba.github.io/page-agent/docs/introduction/overview/)
- [PageAgent GitHub](https://github.com/alibaba/page-agent)
- [Ollama 官網](https://ollama.com)
