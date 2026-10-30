# 儲存

本頁說明 SQLite 的三張資料表與 `database.SQLite` 的存取方法；檔案位置規則見 [設定](/zh/configuration)。

## 資料表

| 資料表 | 欄位 | 用途 |
|--------|------|------|
| `news` | `id`、`title`、`url`（UNIQUE）、`content`、`full_content`、`source`、`author`、`word_count`、`published_at`、`created_at` | 文章與萃取全文 |
| `feeds` | `id`、`url`（UNIQUE）、`dismiss`、`created_at`、`updated_at` | 訂閱源；`dismiss = 1` 為已移除 |
| `data` | `id`、`key`（UNIQUE）、`value` | 鍵值設定：`apikey`、`summary` |

`NewSQLite()` 啟動時以 `CREATE TABLE IF NOT EXISTS` 建表，並在 `news.url`、`news.published_at`、`news.source`、`feeds.url`、`feeds.dismiss`、`data.key` 建立索引。

## 文章

| 方法 | 行為 |
|------|------|
| `Insert(news, content)` | `INSERT OR REPLACE`；`content` 為 nil 時全文、作者為空字串、字數為 0 |
| `Get(hours)` | 回傳 `published_at` 在 N 小時內的文章，由新到舊；單列讀取失敗時略過該列 |
| `GetFromURL(url)` | 以 URL 取單篇；查無資料時回傳錯誤 |

讀出時空字串的全文與作者、為 0 的字數轉為 nil，呼叫端以 `FullContent == nil` 判斷是否需要萃取。

## 訂閱源

| 方法 | 行為 |
|------|------|
| `InsertFeed(url)` | `INSERT OR REPLACE`，`dismiss` 設 0；重新加入的訂閱源建立時間會更新 |
| `RemoveFeed(url)` | 將 `dismiss` 設 1 並更新 `updated_at` |
| `GetFeed()` | 回傳 `dismiss = 0` 的 URL，依建立時間排序 |

## 鍵值與連線

| 方法 | 行為 |
|------|------|
| `GetKey(key)` | 取值；不存在時回傳錯誤 |
| `SetKey(key, value)` | `INSERT OR REPLACE`，鍵與值皆去除前後空白 |
| `Close()` | 關閉連線；`Run()` 結束時呼叫 |
