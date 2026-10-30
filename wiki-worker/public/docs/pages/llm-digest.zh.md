# LLM 新聞概要

本頁說明 Summary 欄的新聞概要何時產生、送出哪些內容，以及 `internal/api` 的串流用戶端。

## 觸發條件

| 條件 | 結果 |
|------|------|
| 一次檢查發現至少 1 篇新文章 | 全文萃取完成後產生概要 |
| 沒有新文章 | 不產生，Summary 保留上次結果 |
| 啟動時 | 從資料庫 `summary` 載入上次結果，不呼叫 API |

概要請求在 tview 主執行緒（`QueueUpdateDraw`）內執行，等待回應期間介面不會回應按鍵。

## 送出內容

| 角色 | 內容 |
|------|------|
| `system` | 固定的繁體中文指令、當下時間、作業系統／架構，以及前次概要 |
| `user`（僅無前次概要時） | 資料庫中 24 小時內的每篇文章 |
| `user` | 本次抓取的每篇文章（72 小時內） |

每篇文章送出 `Title`、`Source`、`PublishedAt`（本地時間）與 `Content`；`Content` 是 RSS 描述，不是萃取後的全文。

## 輸出結構

system 指令要求以繁體中文、依下列順序輸出：

| 段落 | 內容 |
|------|------|
| 重大要聞 | 國際局勢、政府政策、重大社會事件 |
| 科技與金融 | 科技趨勢、市場動向、經濟指標 |
| 生活資訊 | 天氣、交通、消費、健康、教育文化 |
| 趨勢分析 | 與前次概要比對的變化與後續發展 |

成功時結果寫回 `data` 表的 `summary` 並顯示；失敗時錯誤訊息顯示在 Summary 欄，資料庫中的舊概要不變。

## API 用戶端

| 符號 | 行為 |
|------|------|
| `api.ApiKey` | 套件層級變數；產生概要前從資料庫 `apikey` 載入 |
| `AskWithSmallModel(msgList)` | 以 `gpt-4o-mini` 呼叫；不檢查金鑰 |
| `AskWithLargeModel(msgList)` | 以 `gpt-4o` 呼叫；金鑰為空或短於 20 字元時回傳 `API key is not set`；目前介面未使用 |
| `Message` | `Role`、`Content` |
| `Request` | `Model`、`Messages`、`Stream`（固定 `true`） |
| `Response` | 串流區塊的 `Choices[].Delta.Content` |

請求送往 `https://api.openai.com/v1/chat/completions`，以 SSE 逐行讀取 `data:` 區塊並串接 `delta.content`；非 200 回應回傳 `API Error (Status N): <body>`。HTTP 用戶端沒有設定逾時。
