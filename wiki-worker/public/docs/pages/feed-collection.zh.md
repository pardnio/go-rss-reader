# 訂閱源收集

本頁說明 `Collector` 如何管理訂閱源，以及如何把 RSS 轉成新聞列表。

## 訂閱源管理

| 方法 | 對應指令 | 行為 |
|------|----------|------|
| `NewCollector(db)` | — | 建立收集器，HTTP 逾時 30 秒 |
| `Add(link)` | `add` | 空字串直接返回；否則 `InsertFeed()`，已存在的 URL 會重設為啟用 |
| `Remove(link)` | `rm`／`remove` | `RemoveFeed()`，將 `dismiss` 設為 1 |
| `List()` | `config` | 回傳啟用中的訂閱源，依建立時間排序 |

## 抓取與解析

`GetNews()` 逐一處理啟用中的訂閱源：

| 步驟 | 行為 |
|------|------|
| 抓取 | HTTP GET 後以 `encoding/xml` 解析為 `model.RSS`；失敗只寫 log 並略過該訂閱源 |
| 來源名稱 | 取 `<channel><title>`；為空時用訂閱源 URL |
| 去重 | 以去除空白後的 `<link>` 為鍵，跨所有訂閱源去重 |
| 時間窗 | 發布時間早於 72 小時前的項目略過 |
| 清理 | 標題與描述移除 HTML 標籤並還原 HTML entity |
| 排序 | 依發布時間由新到舊 |

每個回應內容都依 RSS 2.0 的 `<rss><channel><item>` 結構解碼。不檢查 HTTP 狀態碼：根元素不是 `<rss>` 的內容（例如 Atom 的 `<feed>` 或 HTML 錯誤頁）解碼失敗，該訂閱源在本次檢查中略過。

## 日期解析

`pubDate` 依序嘗試下列格式，全部轉為 UTC：

| 格式 |
|------|
| `Mon,02 Jan 2006 15:04:05 -0700` |
| `Mon, 02 Jan 2006 15:04:05 -0700` |
| `Mon, 2 Jan 2006 15:04:05 -0700` |
| `2006-01-02T15:04:05Z07:00` |
| `2006-01-02 15:04:05` |
| `time.RFC1123Z`、`time.RFC1123`、`time.RFC3339` |

`pubDate` 為空或無法解析時使用當下時間；[刷新流程](/zh/refresh-cycle) 會在文章已入庫時改回資料庫中的時間。

## 資料模型

| 型別 | 欄位 |
|------|------|
| `model.RSS` | `Channel` |
| `model.Channel` | `Title`、`Description`、`Item []Item` |
| `model.Item` | `Title`、`Description`、`Link`、`PubDate` |
| `model.News` | `Title`、`Content`（RSS 描述）、`Source`、`URL`、`PublishedAt`、`FullContent`、`Author`、`WordCount`（後三者為指標，未萃取時為 nil） |
