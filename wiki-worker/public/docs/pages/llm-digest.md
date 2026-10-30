# LLM Digest

This page explains when the news digest in the Summary pane is generated, what is sent, and the streaming client in `internal/api`.

## Trigger Conditions

| Condition | Result |
|-----------|--------|
| A check finds at least one new article | Generate a digest after full-text extraction finishes |
| No new articles | No digest; Summary keeps the last result |
| On launch | Load the last result from the `summary` key without calling the API |

The digest request runs on the tview main thread (`QueueUpdateDraw`), so the interface does not respond to keys while waiting for the reply.

## What Gets Sent

| Role | Content |
|------|---------|
| `system` | Fixed Traditional Chinese instructions, the current time, OS/architecture, and the previous digest |
| `user` (only without a previous digest) | Every article stored in the last 24 hours |
| `user` | Every article from this fetch (within 72 hours) |

Each article sends `Title`, `Source`, `PublishedAt` (local time), and `Content`; `Content` is the RSS description, not the extracted full text.

## Output Structure

The system instructions ask for Traditional Chinese output in this order:

| Section | Content |
|---------|---------|
| 重大要聞 (major news) | International affairs, government policy, major social events |
| 科技與金融 (tech & finance) | Tech trends, market moves, economic indicators |
| 生活資訊 (lifestyle) | Weather, traffic, consumer, health, education and culture |
| 趨勢分析 (trend analysis) | Changes against the previous digest and likely follow-ups |

On success the result is written back to `summary` in the `data` table and displayed; on failure the error message appears in Summary and the stored digest stays unchanged.

## API Client

| Symbol | Behavior |
|--------|----------|
| `api.ApiKey` | Package-level variable; loaded from `apikey` in the database before a digest |
| `AskWithSmallModel(msgList)` | Calls `gpt-4o-mini`; does not check the key |
| `AskWithLargeModel(msgList)` | Calls `gpt-4o`; returns `API key is not set` when the key is empty or shorter than 20 characters; unused by the interface |
| `Message` | `Role`, `Content` |
| `Request` | `Model`, `Messages`, `Stream` (always `true`) |
| `Response` | `Choices[].Delta.Content` of a stream chunk |

Requests go to `https://api.openai.com/v1/chat/completions`; `data:` lines are read from the SSE stream and `delta.content` is concatenated. A non-200 response returns `API Error (Status N): <body>`. The HTTP client has no timeout.
