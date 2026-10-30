# Storage

This page explains the three SQLite tables and the access methods on `database.SQLite`; see [Configuration](/configuration) for where the file lives.

## Tables

| Table | Columns | Purpose |
|-------|---------|---------|
| `news` | `id`, `title`, `url` (UNIQUE), `content`, `full_content`, `source`, `author`, `word_count`, `published_at`, `created_at` | Articles and extracted text |
| `feeds` | `id`, `url` (UNIQUE), `dismiss`, `created_at`, `updated_at` | Feeds; `dismiss = 1` means removed |
| `data` | `id`, `key` (UNIQUE), `value` | Key-value settings: `apikey`, `summary` |

On startup `NewSQLite()` creates the tables with `CREATE TABLE IF NOT EXISTS` and indexes `news.url`, `news.published_at`, `news.source`, `feeds.url`, `feeds.dismiss`, and `data.key`.

## Articles

| Method | Behavior |
|--------|----------|
| `Insert(news, content)` | `INSERT OR REPLACE`; with a nil `content`, full text and author are empty strings and word count is 0 |
| `Get(hours)` | Returns articles whose `published_at` is within N hours, newest first; rows that fail to scan are skipped |
| `GetFromURL(url)` | Fetches one article by URL; returns an error when not found |

On read, empty full text and author and a zero word count become nil, so callers check `FullContent == nil` to decide whether to extract.

## Feeds

| Method | Behavior |
|--------|----------|
| `InsertFeed(url)` | `INSERT OR REPLACE` with `dismiss` set to 0; a re-added feed gets a new creation time |
| `RemoveFeed(url)` | Sets `dismiss` to 1 and updates `updated_at` |
| `GetFeed()` | Returns URLs with `dismiss = 0`, ordered by creation time |

## Key-Value and Connection

| Method | Behavior |
|--------|----------|
| `GetKey(key)` | Reads a value; returns an error when missing |
| `SetKey(key, value)` | `INSERT OR REPLACE`, trimming whitespace from key and value |
| `Close()` | Closes the connection; called when `Run()` returns |
