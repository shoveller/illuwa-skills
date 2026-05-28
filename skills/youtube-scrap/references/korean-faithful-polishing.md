# Korean faithful polishing for lecture notes

Use this reference when a YouTube/Web lecture note is written in Korean and a polishing skill such as `yoonmoon` is available.

## Purpose

Polishing is a readability pass, not a second summary. The goal is to make Korean sentences natural while preserving the source's chronology, stance, and evidence.

## Preserve exactly

- Timestamp values and timestamp basis labels.
- Source facts, names, product names, commands, paths, URLs, warnings, and caveats.
- YAML frontmatter keys and values unless the user explicitly asked for metadata edits.
- Heading hierarchy and table-of-contents order, especially when timestamps are used as the outline.
- Speaker nuance: required vs optional, recommended vs rejected, safe vs risky, beginner-friendly vs advanced.

## Polish actively

- Convert obvious passive or translated phrasing into active Korean when the actor or mechanism is clear.
  - `문서에서 사용된다` → `문서에서 사용한다`
  - `타임스탬프가 목차로 사용된다` → `타임스탬프를 목차로 삼는다`
- Smooth rough operational wording without weakening technical meaning.
  - `프로세스가 죽는다` → `프로세스가 멈춘다`
  - `포트를 잡는다` → `포트를 사용한다` / `포트를 연결한다`
- Prefer short, dry technical-blog sentences over translated English essay framing.
  - Avoid bloated frames like `~라는 관점에서`, `문제의식`, `실생활에서 유용하다`, `충분하지 않다` unless they are directly source-significant.
  - Use direct forms like `핵심은 X이다`, `X만으로는 Y하기 어렵다`, `그래서 Z가 필요하다`.
- Expand overly dense Sino-Korean compounds when that improves legibility without changing meaning.
  - `고권한` / `고권` → `높은 권한`

## Final self-check

Before saving or replying, compare the polished version against the pre-polish draft:

1. No timestamp, URL, command, product name, or metadata value changed accidentally.
2. No warning became softer or stronger than the source.
3. No rejected option became a recommendation.
4. No invented timestamp or quotation appeared during polishing.
5. The Korean reads naturally without essay-like padding.
