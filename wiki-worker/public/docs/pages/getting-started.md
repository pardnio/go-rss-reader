# Getting Started

This page walks through building go-rss-reader from source, adding the first feed, and reading the first full article.

## Prerequisites

| Item | Requirement |
|------|-------------|
| Go | 1.24.3 or higher (`go.mod`) |
| C compiler | gcc or clang with `CGO_ENABLED=1`; `github.com/mattn/go-sqlite3` is a CGO package |
| Terminal | Color support |
| OpenAI API key | Optional; only the LLM news digest needs it |
| Open original article | `open` on macOS, `xdg-open` on Linux, `cmd /c start` on Windows |

## Installation

```bash
git clone https://github.com/pardnchiu/go-rss-reader.git
cd go-rss-reader
CGO_ENABLED=1 go build -o RSSReader ./cmd/cli
```

The module name in `go.mod` is `rss-reader`, not the GitHub path, so `go install github.com/pardnchiu/go-rss-reader/...` does not work; build from source or run directly with `go run ./cmd/cli`.

## First Run

```bash
./RSSReader
```

1. The app starts with no feeds. Press Tab twice to move focus from **News List** through **Preview** to **Command**
2. Type the command below and press Enter:

   ```text
   add https://feeds.bbci.co.uk/news/rss.xml
   ```

3. Press `Ctrl+R` to fetch now; once the status bar shows `Found N new articles, getting full content...`, full-text extraction runs in the background
4. Press Tab twice (**Command** → **Summary** → **News List**) to return to **News List** and move with `↑`/`↓`; **Preview** shows the extracted full text
5. Press `Ctrl+O` to open the original article in the default browser, and `Ctrl+C` to quit

When run inside the project directory (the working directory has `go.mod`), the database is created at `./rss.db`; see [Configuration](/configuration) for the other locations.

## Enable the LLM Digest

Type in the **Command** pane:

```text
apikey sk-xxxxxxxxxxxxxxxxxxxxxxxx
```

The next time new articles arrive and full-text extraction finishes, the **Summary** pane shows a news digest in Traditional Chinese. See [LLM Digest](/llm-digest) for the trigger conditions and what gets sent.
