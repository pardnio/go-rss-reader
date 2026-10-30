# Refresh Cycle

This page explains the full order of one update check: trigger, new-article detection, background full-text extraction, and digest generation.

## Triggers

| Source | Call | Status bar |
|--------|------|------------|
| Launch | `Run()` → `getList(false)` | `Loading...` |
| `Ctrl+R` | `getList(true)` | `Checking...` |
| Auto check | 5-minute ticker → `getList(true)` | `Refreshing...` → `Checking...` |

The auto check starts as soon as `New()` creates the ticker and has no off switch; `stopRefresh()` stops the ticker when the program exits.

## Check Steps

`getList()` runs these steps in a background goroutine:

| Step | Action |
|------|--------|
| 1 | If the list is empty, show articles from the last 72 hours via `database.Get(72)` first |
| 2 | `Collector.GetNews()` fetches every active feed and returns articles within 72 hours |
| 3 | Look up each article with `GetFromURL()`: not found counts as new; found reuses the stored publish time |
| 4 | Sort newest first and update News List |
| 5 | When there are new articles, run `loadContent()` in another goroutine |

Step 3 reuses the stored time because an RSS item without a parseable `pubDate` falls back to the current time, which would drift on every check.

Step 4 replaces the list with what this fetch returned: an article that has dropped out of every feed leaves News List even though it stays in the database, and when no feed returns anything the list is emptied and the status bar shows `All news are up to date.`

## Background Full-Text Extraction

`loadContent()` handles each article from this fetch:

| Condition | Action |
|-----------|--------|
| Full text already stored | Skip |
| Extraction succeeds | Store the article and full text, update progress, wait 500ms before the next one |
| Extraction fails | Store the article with empty full text, log it, and move on immediately |

Articles with empty full text are retried the next time a check finds at least one new article, because `loadContent()` walks every article from that fetch. When every article is processed, the [LLM Digest](/llm-digest) runs.

## Extraction on Preview

Moving through News List triggers `showPreview()` for a single article:

| Condition | Display |
|-----------|---------|
| Full text already stored | Show the full text directly |
| No stored full text, live extraction succeeds | Show the full text and store it in the background |
| Live extraction fails | Show the RSS description |
