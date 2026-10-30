# 設定

本頁列出 go-rss-reader 的所有設定來源：兩個環境變數、資料庫檔案位置，以及存在資料庫內的 API 金鑰。

## 環境變數

| 變數 | 必要 | 說明 |
|------|------|------|
| `RSS_DB_PATH` | 否 | SQLite 資料庫檔案完整路徑；有值時跳過所有自動判斷，上層目錄必須已存在 |
| `GO_ENV` | 否 | 值為 `development` 時視為開發環境，資料庫建立在目前工作目錄 |

程式沒有設定檔，也不讀 `.env`。

## 資料庫位置

`database.NewSQLite()` 依下列順序決定 `rss.db` 的路徑：

| 順序 | 條件 | 路徑 |
|------|------|------|
| 1 | `RSS_DB_PATH` 有值 | 該值 |
| 2 | 目前工作目錄存在 `go.mod`，或 `GO_ENV=development` | `<工作目錄>/rss.db` |
| 3 | 執行檔所在目錄可寫入（以建立再刪除 `.write_test` 測試） | `<執行檔目錄>/rss.db` |
| 4 | 其他 | `~/.rss-reader/rss.db`，目錄不存在時以 `0755` 建立 |

資料庫初始化失敗時程式以 `Failed to init SQLite` 結束，不會進入介面。

## API 金鑰

| 項目 | 行為 |
|------|------|
| 設定方式 | 介面 **Command** 欄輸入 `apikey <API_KEY>` |
| 儲存位置 | `data` 資料表，key 為 `apikey`，明文儲存 |
| 載入時機 | 每次全文萃取完成、準備產生概要時讀取 |
| 顯示 | `config` 指令會在 Preview 以明文顯示金鑰，未設定時顯示 `Not set`；沒有訂閱源時只顯示 `No RSS feeds available.`，不顯示金鑰 |

金鑰不讀環境變數。未設定金鑰時仍會送出概要請求，OpenAI 回傳的 401 錯誤訊息會顯示在 **Summary** 欄。

## 固定參數

下列數值寫死在原始碼，沒有對應的設定項：

| 項目 | 值 | 位置 |
|------|-----|------|
| 自動檢查間隔 | 5 分鐘 | `internal/app/app.go` `New()` |
| 新聞保留視窗 | 72 小時 | `internal/util/collector.go` `GetNews()` |
| HTTP 逾時（RSS、全文） | 30 秒 | `NewCollector()`／`NewExtractor()` |
| 全文萃取間隔 | 500ms | `loadContent()` |
| 概要模型 | `gpt-4o-mini` | `internal/api/llm.go` |
