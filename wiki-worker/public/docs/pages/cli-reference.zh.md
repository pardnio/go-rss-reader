# 命令列參考

本頁列出 go-rss-reader 的啟動方式、快捷鍵、介面內指令與各介面區塊的內容。

## 啟動

| 指令 | 說明 |
|------|------|
| `./RSSReader` | 啟動 TUI；沒有 flag 也沒有子命令 |
| `go run ./cmd/cli` | 不建置直接執行 |

資料庫無法開啟時程式在介面出現前以 `Failed to init SQLite: <err>` 結束；介面本身啟動失敗時輸出 `Failed to launch: <err>`。

## 快捷鍵

快捷鍵在應用程式層攔截，任何焦點下都有效。

| 按鍵 | 說明 |
|------|------|
| Tab | 依序切換焦點：News List → Preview → Command → Summary → News List |
| `Ctrl+R` | 立即檢查所有訂閱源 |
| `Ctrl+O` | 以預設瀏覽器開啟 News List 目前選取的新聞 |
| `↑`／`↓` | 在 News List 移動；在 Preview／Summary 捲動 |
| Enter | 在 Command 欄執行指令 |
| `Ctrl+C` | 結束程式 |

## 介面內指令

指令名稱不分大小寫，參數以空白分隔，只取第一個參數。

| 指令 | 語法 | 行為 |
|------|------|------|
| `add` | `add <URL>` | 新增訂閱源後在 Preview 列出訂閱源；已移除的 URL 會重新啟用 |
| `rm`／`remove` | `rm <URL>` | 將訂閱源標記為已移除（`dismiss = 1`），已存的文章保留；未訂閱的 URL 不會有任何變動 |
| `apikey` | `apikey <API_KEY>` | 將 OpenAI 金鑰寫入資料庫 |
| `config` | `config` | 在 Preview 顯示 API 金鑰與訂閱源列表；沒有訂閱源時只顯示 `No RSS feeds available.` |

缺少參數時 Preview 顯示用法（如 `add [URL]`）；無法辨識的指令顯示 `未知指令: <指令>`。

## 介面區塊

| 區塊 | 位置 | 內容 |
|------|------|------|
| 狀態列 | 頂端 | 目前狀態、`Next check at HH:MM`（最近一次狀態更新後 5 分鐘）、`Ctrl+R`／`Ctrl+O` 提示 |
| Summary | 左上 | LLM 新聞概要；啟動時載入上次結果 |
| News List | 左下 | 72 小時內新聞，依發布時間由新到舊；副標為 `MM/DD HH:mm \| 來源` |
| Preview | 右上 | 標題、作者、來源、發布時間、字數、連結與全文；萃取失敗時改顯示 RSS 描述 |
| Command | 右下 | 指令輸入欄 |

## 狀態訊息

| 訊息 | 意義 |
|------|------|
| `Loading...` | 啟動時第一次抓取 |
| `Refreshing...` | 5 分鐘自動檢查觸發，隨即換成 `Checking...` |
| `Checking...` | `Ctrl+R` 或自動檢查 |
| `Get N news from Database` | 列表為空時先顯示資料庫快取 |
| `Found N new articles, getting full content...` | 發現新文章，背景萃取全文 |
| `Progress: X% (i/n)` | 全文萃取進度 |
| `All news are up to date.` | 沒有新文章 |
| `All news are up to date` | 背景全文萃取完成，接著送出 LLM 概要請求 |
| `Failed to get news list: <err>` | 讀取訂閱源列表失敗 |
