# CLI Reference

This page lists how to launch go-rss-reader, its hotkeys, in-app commands, and what each pane shows.

## Launch

| Command | Description |
|---------|-------------|
| `./RSSReader` | Start the TUI; takes no flags or subcommands |
| `go run ./cmd/cli` | Run without building |

When the database cannot be opened the program exits with `Failed to init SQLite: <err>` before the interface appears; when the interface itself fails it prints `Failed to launch: <err>`.

## Hotkeys

Hotkeys are captured at the application level and work regardless of focus.

| Key | Description |
|-----|-------------|
| Tab | Cycle focus: News List → Preview → Command → Summary → News List |
| `Ctrl+R` | Check all feeds now |
| `Ctrl+O` | Open the article selected in News List in the default browser |
| `↑`/`↓` | Move in News List; scroll in Preview/Summary |
| Enter | Run the command in the Command pane |
| `Ctrl+C` | Quit |

## In-App Commands

Command names are case-insensitive; arguments are space-separated and only the first is used.

| Command | Syntax | Behavior |
|---------|--------|----------|
| `add` | `add <URL>` | Add a feed, then list feeds in Preview; a removed URL is re-enabled |
| `rm`/`remove` | `rm <URL>` | Mark the feed as removed (`dismiss = 1`); stored articles stay; a URL that is not subscribed changes nothing |
| `apikey` | `apikey <API_KEY>` | Write the OpenAI key to the database |
| `config` | `config` | Show the API key and feed list in Preview; with no feeds it only shows `No RSS feeds available.` |

A missing argument shows the usage in Preview (such as `add [URL]`); an unknown command shows `未知指令: <command>`.

## Panes

| Pane | Position | Content |
|------|----------|---------|
| Status bar | Top | Current status, `Next check at HH:MM` (five minutes after the latest status update), `Ctrl+R`/`Ctrl+O` hints |
| Summary | Top left | LLM news digest; the last result loads on start |
| News List | Bottom left | News from the last 72 hours, newest first; subtitle `MM/DD HH:mm \| source` |
| Preview | Top right | Title, author, source, publish time, word count, link, and full text; falls back to the RSS description when extraction fails |
| Command | Bottom right | Command input |

## Status Messages

| Message | Meaning |
|---------|---------|
| `Loading...` | First fetch on start |
| `Refreshing...` | The 5-minute auto check fired; replaced by `Checking...` right after |
| `Checking...` | `Ctrl+R` or auto check |
| `Get N news from Database` | Cached articles shown first while the list is empty |
| `Found N new articles, getting full content...` | New articles found; extracting full text in the background |
| `Progress: X% (i/n)` | Full-text extraction progress |
| `All news are up to date.` | No new articles |
| `All news are up to date` | Background extraction finished; the LLM digest is requested next |
| `Failed to get news list: <err>` | Reading the feed list failed |
