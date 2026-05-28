# NotebookLM YouTube scrap grounding pattern

Use this note when a YouTube lecture is being scraped with NotebookLM and the normal transcript path is unreliable.

## Pattern observed

- NotebookLM can accept a YouTube source and produce a source-grounded chronology even when the MCP tool returns sparse `sources`/`sourceIds` in the later analysis response.
- Public transcript extraction may fail because the execution IP is blocked by YouTube, and `yt-dlp` may also hit bot/sign-in checks.
- In that case, do not discard the NotebookLM result if NotebookLM itself clearly analyzed the YouTube source. Use it, but label timestamp grounding precisely.

## Required output behavior

1. Normalize the YouTube URL to `https://www.youtube.com/watch?v=<id>` for the NotebookLM source and final metadata.
2. Cross-check basic metadata with a low-risk public endpoint such as YouTube oEmbed when available: title, channel, thumbnail/provider.
3. Try transcript extraction once or twice through the standard skill path. If it fails due to YouTube blocking, record that as a limitation, not as a reason to invent timestamps.
4. If NotebookLM returns chronological content but no verified MM:SS values, use a source-order table of contents and explicitly write: `정확한 MM:SS 타임스탬프 확인 불가`.
5. Preserve the lecture's sequence and warning strength. Do not convert a security warning into a casual caveat.
6. If saving to KiwiFS, include frontmatter fields for `source_url`, `source_title`, `source_author`, `notebooklm_url`, `notebooklm_id`, and `timestamp_basis`.
7. Run wiki lint and a page health/read-back check before reporting completion.

## Pitfall

A successful NotebookLM chronology is not the same as a verified timestamp transcript. It is acceptable as a lecture-flow source, but exact MM:SS headings require transcript, chapter, or description timestamp evidence.