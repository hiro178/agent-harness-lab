# Source Routing — Fetcher Selection

How to pick a fetch strategy for a given source in Phase 1a.

## Routing table

| Priority | Source type | Fetch strategy | Reason |
|----------|-------------|-----------------|--------|
| 1 | JS-rendered social/dynamic pages | A browser-capable fetch, or a dedicated tool if one is available for that platform | Plain HTTP fetch often returns an empty shell |
| 2 | Video with transcript/captions | Transcript extraction, then treat the transcript as the source text | The transcript, not the video metadata, is the content |
| 3 | RSS/Atom feed | Feed parser producing structured entries | Avoids re-parsing HTML for content already structured |
| 4 | PDF | Direct download + page-range read | Generic web-fetch tools often can't handle binary PDF content |
| 5 | Long-form academic paper (arXiv, ACM, IEEE...) | Direct download + read, not a summarizing fetch | Long technical content is exactly where summarization bias does the most damage |
| 6 | Long-form blog post (roughly >5000 words) | Direct download + read | Same reasoning as above — depth matters more than fetch cost |
| 7 | Generic web page (none of the above) | Standard web-fetch tool | Default path, lowest cost |

## Downgrade triggers

Even after picking a strategy, downgrade to a direct-read approach when any of these fire:

- The fetched summary is under ~500 characters for a source that promised depth.
- The summary uses meta-phrases ("the article discusses...") instead of quoting actual content.
- Phase 2 extraction confidence comes back low across all candidate items — retry with a deeper fetch.

## Extension protocol

Adding a new source pattern: add one row to this table (priority number controls ordering). SKILL.md needs no change — it only links here. If no dedicated tool exists for a pattern, route to the generic direct-read or standard-fetch path.
