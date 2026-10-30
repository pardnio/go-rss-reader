# Content Extraction

This page explains how `Extractor` pulls the title, author, and body out of a news page, and how the word count is calculated.

## Extraction Flow

`NewExtractor()` creates an extractor with a 30-second HTTP timeout; `Get(url)` fetches the page and parses it with goquery:

| Step | Rule |
|------|------|
| Remove noise | Drop `script`, `style`, `nav`, `aside`, `footer`, `.advertisement`, `.ads`, `.comment` |
| Title | The first `<h1>`; falls back to `<title>` |
| Author | Text of the first element matching `[rel='author']`, `.author`, or `[itemprop='author']` |
| Body | See the next section |
| Normalize | Collapse runs of whitespace into a single space |

It returns `model.NewsContent{Title, Author, Content, WordCount}`; on an HTTP or HTML parse error it returns the error and the caller falls back to the RSS description. The HTTP status code is not checked, so a 404 or paywall page is parsed like any other page.

The extracted `Title` is neither displayed nor stored: Preview and the `news` table keep the RSS title.

## Body Selection

Tried in order; the first match wins:

| Order | Condition |
|-------|-----------|
| 1 | Combined text of every `<article>` element, longer than 128 bytes |
| 2 | Combined text of every `<main>` element, longer than 128 bytes |
| 3 | Among all `<div>` elements, the longest one over 128 bytes whose link text is under 30% of its text |

The link ratio in step 3 rules out link-heavy blocks such as navigation bars and related-article lists. When none match, the body is an empty string. An empty body is stored as empty full text, which reads back as nil, so the article is extracted again the next time it is previewed or the next background extraction runs.

## Word Count

`WordCount` is the number of Han characters (`\p{Han}`) plus the number of English words (`[a-zA-Z]+`), counted on the body before normalization.
