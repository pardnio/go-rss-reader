# 快速開始

本頁說明從原始碼建置 go-rss-reader、加入第一個訂閱源並讀到第一篇全文的完整步驟。

## 前置需求

| 項目 | 需求 |
|------|------|
| Go | 1.24.3 以上（`go.mod`） |
| C 編譯器 | gcc 或 clang，且 `CGO_ENABLED=1`；`github.com/mattn/go-sqlite3` 為 CGO 套件 |
| 終端機 | 支援色彩顯示 |
| OpenAI API 金鑰 | 選用；只有 LLM 新聞概要需要 |
| 開啟原文 | macOS `open`、Linux `xdg-open`、Windows `cmd /c start` |

## 安裝

```bash
git clone https://github.com/pardnchiu/go-rss-reader.git
cd go-rss-reader
CGO_ENABLED=1 go build -o RSSReader ./cmd/cli
```

`go.mod` 的 module 名稱為 `rss-reader`，不是 GitHub 路徑，因此 `go install github.com/pardnchiu/go-rss-reader/...` 無法使用，只能從原始碼建置或以 `go run ./cmd/cli` 直接執行。

## 第一次執行

```bash
./RSSReader
```

1. 啟動後沒有任何訂閱源。按兩次 Tab，焦點會從 **News List** 經 **Preview** 移到 **Command**
2. 輸入下列指令並按 Enter：

   ```text
   add https://feeds.bbci.co.uk/news/rss.xml
   ```

3. 按 `Ctrl+R` 立即抓取；狀態列顯示 `Found N new articles, getting full content...` 後，背景開始逐篇萃取全文
4. 按兩次 Tab（**Command** → **Summary** → **News List**）回到 **News List**，以 `↑`／`↓` 移動，**Preview** 會顯示萃取後的全文
5. 按 `Ctrl+O` 以預設瀏覽器開啟原文，`Ctrl+C` 結束程式

在專案目錄內執行時（工作目錄有 `go.mod`），資料庫會建立在 `./rss.db`；其他位置的判斷規則見 [設定](/zh/configuration)。

## 啟用 LLM 概要

在 **Command** 欄輸入：

```text
apikey sk-xxxxxxxxxxxxxxxxxxxxxxxx
```

下一次抓到新文章並完成全文萃取後，**Summary** 欄會出現繁體中文新聞概要。觸發條件與送出內容見 [LLM 新聞概要](/zh/llm-digest)。
