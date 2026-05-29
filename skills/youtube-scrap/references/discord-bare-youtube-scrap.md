# Discord bare YouTube scrape pattern

When a Discord user sends only a YouTube URL plus a short Korean verb such as `스크랩`, infer the established wiki workflow unless the user says otherwise.

## Default behavior

- Use NotebookLM as the primary analysis surface when auth is available.
- Search `00 Inbox/youtube` first by video ID and URL to avoid duplicates.
- Save a Korean Markdown note into `00 Inbox/youtube/YYYY-MM-DD-<topic-slug>.md` through KiwiFS.
- Use Korean technical-document plain style: `~한다`, `~이다`, `~된다`.
- Include at least:
  - YAML frontmatter with source URL/title/channel, NotebookLM URL/ID, tags, status, updated date.
  - concise overview.
  - 기능/구성요소 table when the source is a tutorial, tool demo, setup guide, or workflow explanation.
  - source-order detailed notes.
  - 실습 절차, 주의사항, 용어, 한계와 출처.
- If exact MM:SS timestamps are unavailable, state that explicitly and do not invent timestamp headings.
- Run `lint` and usually `health_check`, then report absolute wiki URL plus NotebookLM URL.

## Practical note

If NotebookLM successfully ingests and analyzes the YouTube source, do not block the workflow on separate transcript/metadata extraction. Extra extraction is optional verification, not a prerequisite for saving a useful source-grounded note.