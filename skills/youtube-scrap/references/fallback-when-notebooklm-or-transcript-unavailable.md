# Fallback when NotebookLM or YouTube transcript extraction is unavailable

Use this when a user asks to scrape a YouTube video, but the preferred NotebookLM path is unavailable and direct transcript extraction is blocked.

## Trigger conditions

- NotebookLM auth/status check fails or the source cannot be added.
- `yt-dlp` or transcript extraction fails because YouTube requires sign-in/bot confirmation.
- The user asked for a normal scrape, not a strict transcript-faithful quotation or verbatim timestamped transcript.

## Workflow

1. Do not invent a transcript or pretend NotebookLM ran.
2. Use public metadata and summaries from available surfaces:
   - `web_search` for title, channel, date, description, chapters, links, views/likes when shown;
   - `web_extract` on the YouTube URL when it returns a summary;
   - `web_extract` on official/project links from the video description for contextual verification.
3. Keep real chapter timestamps only when they come from the YouTube description/search result. Label the basis clearly as `YouTube description chapters` or equivalent.
4. If no real timestamps are available, write source-order sections without MM:SS headings.
5. Include a `출처와 한계` / `Sources and Limitations` section that explicitly says:
   - NotebookLM was not used and why, without overgeneralizing that it is broken;
   - transcript extraction was blocked, if applicable;
   - the note relies on public description/chapters, extraction summary, and linked official sources.
6. Still save the wiki note when the destination is clear and the user asked for a scrape. A partial but well-labeled source-grounded note is better than stopping on setup/auth state.
7. Run lint and health/read-back verification as usual.

## What not to capture

Do not add durable claims such as “NotebookLM does not work” or “yt-dlp cannot extract YouTube.” Those are environment/setup states. Capture the fallback pattern and the need to label limitations instead.
