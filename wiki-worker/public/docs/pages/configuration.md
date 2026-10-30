# Configuration

This page lists every configuration source in go-rss-reader: two environment variables, the database file location, and the API key stored in the database.

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `RSS_DB_PATH` | No | Full path to the SQLite database file; when set, all automatic resolution is skipped and the parent directory must already exist |
| `GO_ENV` | No | `development` treats the run as development and creates the database in the current working directory |

There is no config file, and `.env` is not read.

## Database Location

`database.NewSQLite()` resolves the path of `rss.db` in this order:

| Order | Condition | Path |
|-------|-----------|------|
| 1 | `RSS_DB_PATH` is set | That value |
| 2 | `go.mod` exists in the current working directory, or `GO_ENV=development` | `<working dir>/rss.db` |
| 3 | The executable's directory is writable (tested by creating and removing `.write_test`) | `<executable dir>/rss.db` |
| 4 | Otherwise | `~/.rss-reader/rss.db`, creating the directory with `0755` if missing |

If database initialization fails, the program exits with `Failed to init SQLite` before the interface opens.

## API Key

| Item | Behavior |
|------|----------|
| How to set | Type `apikey <API_KEY>` in the **Command** pane |
| Storage | `data` table under key `apikey`, stored in plain text |
| When loaded | Each time full-text extraction finishes and a digest is about to be generated |
| Display | The `config` command shows the key in plain text in Preview, or `Not set` when missing; with no feeds it shows only `No RSS feeds available.` and no key |

The key is not read from the environment. Without a key the digest request is still sent, and the 401 error message from OpenAI appears in the **Summary** pane.

## Fixed Parameters

These values are hard-coded and have no setting:

| Item | Value | Location |
|------|-------|----------|
| Auto-check interval | 5 minutes | `internal/app/app.go` `New()` |
| News window | 72 hours | `internal/util/collector.go` `GetNews()` |
| HTTP timeout (RSS, full text) | 30 seconds | `NewCollector()`/`NewExtractor()` |
| Extraction interval | 500ms | `loadContent()` |
| Digest model | `gpt-4o-mini` | `internal/api/llm.go` |
