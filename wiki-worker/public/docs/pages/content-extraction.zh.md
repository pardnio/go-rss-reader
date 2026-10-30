# 全文萃取

本頁說明 `Extractor` 如何從新聞網頁取出標題、作者與正文，以及字數的計算方式。

## 萃取流程

`NewExtractor()` 建立 HTTP 逾時 30 秒的萃取器；`Get(url)` 抓取網頁後以 goquery 解析：

| 步驟 | 規則 |
|------|------|
| 清除雜訊 | 移除 `script`、`style`、`nav`、`aside`、`footer`、`.advertisement`、`.ads`、`.comment` |
| 標題 | 第一個 `<h1>`；沒有則用 `<title>` |
| 作者 | 第一個符合 `[rel='author']`、`.author`、`[itemprop='author']` 的元素文字 |
| 正文 | 見下節 |
| 正規化 | 連續空白壓縮為單一空白 |

回傳 `model.NewsContent{Title, Author, Content, WordCount}`；HTTP 或 HTML 解析失敗時回傳錯誤，呼叫端改用 RSS 描述。不檢查 HTTP 狀態碼，404 或付費牆頁面同樣照一般網頁解析。

萃取出的 `Title` 不會顯示也不會寫入：Preview 與 `news` 資料表沿用 RSS 標題。

## 正文判定

依序嘗試，取第一個成立者：

| 順序 | 條件 |
|------|------|
| 1 | 所有 `<article>` 元素合併後的文字，長度超過 128 bytes |
| 2 | 所有 `<main>` 元素合併後的文字，長度超過 128 bytes |
| 3 | 所有 `<div>` 中，文字最長、超過 128 bytes，且連結文字佔比低於 30% 者 |

第 3 步的連結佔比用來排除導覽列與相關文章清單這類以連結為主的區塊。三者都不成立時正文為空字串。空正文以空全文寫入資料庫，讀回時為 nil，因此下次預覽該篇或下一次背景萃取時會重新萃取。

## 字數計算

`WordCount` 為中文字（`\p{Han}`）個數加上英文單字（`[a-zA-Z]+`）個數，以正規化前的正文計算。
