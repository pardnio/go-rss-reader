# Feed Collection

This page explains how `Collector` manages feeds and turns RSS into the news list.

## Feed Management

| Method | Command | Behavior |
|--------|---------|----------|
| `NewCollector(db)` | — | Creates a collector with a 30-second HTTP timeout |
| `Add(link)` | `add` | Returns on an empty string; otherwise `InsertFeed()`, re-enabling an existing URL |
| `Remove(link)` | `rm`/`remove` | `RemoveFeed()`, setting `dismiss` to 1 |
| `List()` | `config` | Returns active feeds ordered by creation time |

## Fetch and Parse

`GetNews()` processes each active feed:

| Step | Behavior |
|------|----------|
| Fetch | HTTP GET, then decode into `model.RSS` with `encoding/xml`; on failure, log and skip the feed |
| Source name | `<channel><title>`; falls back to the feed URL when empty |
| Deduplication | Keyed on the trimmed `<link>`, across all feeds |
| Time window | Items published more than 72 hours ago are skipped |
| Cleanup | Strip HTML tags from title and description and unescape HTML entities |
| Sorting | Newest first |

Each response body is decoded as the RSS 2.0 `<rss><channel><item>` structure. The HTTP status code is not checked: a body whose root element is not `<rss>`, such as an Atom `<feed>` or an HTML error page, fails to decode and the feed is skipped for this check.

## Date Parsing

`pubDate` is tried against these layouts in order and converted to UTC:

| Layout |
|--------|
| `Mon,02 Jan 2006 15:04:05 -0700` |
| `Mon, 02 Jan 2006 15:04:05 -0700` |
| `Mon, 2 Jan 2006 15:04:05 -0700` |
| `2006-01-02T15:04:05Z07:00` |
| `2006-01-02 15:04:05` |
| `time.RFC1123Z`, `time.RFC1123`, `time.RFC3339` |

An empty or unparseable `pubDate` uses the current time; the [Refresh Cycle](/refresh-cycle) restores the stored time once the article is in the database.

## Data Model

| Type | Fields |
|------|--------|
| `model.RSS` | `Channel` |
| `model.Channel` | `Title`, `Description`, `Item []Item` |
| `model.Item` | `Title`, `Description`, `Link`, `PubDate` |
| `model.News` | `Title`, `Content` (RSS description), `Source`, `URL`, `PublishedAt`, `FullContent`, `Author`, `WordCount` (the last three are pointers, nil until extracted) |
