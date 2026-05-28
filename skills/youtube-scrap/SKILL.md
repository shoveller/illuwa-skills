---
name: youtube-scrap
description: Use when asked to investigate a lecture-style YouTube video or web page with NotebookLM, preserve the source nuance, organize the result chronologically with timestamps as the outline where available, and optionally save a frontmatter-bearing wiki document into a user-specified Markdown vault.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [scrap, notebooklm, youtube, web, markdown-vault, wiki, timeline]
    related_skills: [youtube-content, kiwifs]
---

# YouTube Scrap

## Overview

`youtube-scrap` is a general-purpose workflow for turning a lecture-style YouTube video or web page into a source-faithful, chronology-first note. Use NotebookLM as the primary investigation surface when available, then write a clean Markdown artifact that preserves the source's meaning, emphasis, warnings, and order.

The core principle is **transmission before compression**. Do not flatten the speaker's stance into a generic summary. Preserve the original problem framing, comparison tone, warning strength, sequence of examples, and practical emphasis.

## When to Use

Use this skill when the user asks to:

- investigate a YouTube lecture, tutorial, talk, course clip, livestream replay, or long-form web page;
- use NotebookLM to analyze a source;
- organize the result by timeline or timestamp;
- preserve the source nuance rather than rewrite it as a generic explainer;
- save the result as a Markdown/wiki document;
- place the result into a specified Markdown vault, Obsidian vault, KiwiFS wiki, docs folder, or similar knowledge base.

Do not use this skill for:

- short one-line summaries where no source-grounded structure is needed;
- code debugging or implementation work unrelated to source investigation;
- private credentials, paywalled material, or sources the agent is not authorized to access.

## Inputs to Identify

Before writing the final artifact, identify as many of these as possible:

- source URL;
- source type: YouTube, web page, PDF-like web document, course page, transcript page;
- title, author/channel/site, publish date if available;
- desired output language and tone;
- requested destination, if any;
- whether the user named a Markdown vault or wiki path;
- whether the user specifically requested timestamps as the table of contents.

If a destination is obvious from the user's request, act on it. Ask only when the target vault/path is genuinely ambiguous and cannot be inferred from the current workspace or connected wiki tools.

## Workflow

### 1. Normalize and inspect the source

1. Normalize the URL.
   - Convert YouTube short links, `/live/` URLs, tracking-query URLs, and embeds to a canonical source URL when possible.
2. Capture basic metadata.
   - Title, channel/author/site, source URL, and any public description or chapter/timestamp list.
3. Look for existing notes before creating a new one.
   - Search by URL, video ID, title, author, and key topic terms in the target vault/wiki when one is available.

### 2. Investigate with NotebookLM

1. Check NotebookLM availability/auth before creating side effects.
2. Create or reuse a notebook appropriate to the source.
3. Add the source as a YouTube or web source.
4. Ask NotebookLM for a source-grounded chronology-first analysis.
5. Require NotebookLM to distinguish real timestamps from inferred sequence.
6. Remove NotebookLM wrapper text from the final artifact.

Suggested NotebookLM task shape:

```text
Analyze this source for a Markdown wiki note. Preserve the source's chronology, intent, emphasis, warnings, and practical nuance. If exact timestamps or chapters are present, use them as the outline. If exact timestamps are not available, do not invent them; state that exact timestamps are unavailable and keep the sequence in source order. Include: source metadata, concise overview, timestamp/chronology table of contents, section-by-section notes, practical steps if present, warnings/caveats, terms, and source URL.
```

### 3. Preserve timestamp grounding

Use the strongest available timestamp evidence, in this order:

1. source transcript with timestamps;
2. YouTube chapter list or description timestamps;
3. NotebookLM citations/timeline that explicitly include MM:SS;
4. source-order sections without exact timestamps.

Rules:

- Never invent MM:SS values.
- If using YouTube description chapters, label them as description/chapter timestamps.
- If transcript extraction fails or is blocked, state the limitation in the artifact.
- If only source-order chronology is available, write headings without fabricated times.

### 4. Write the Markdown artifact

Use a stable, reader-facing structure. Adapt heading names to the user's language.

Recommended shape:

```markdown
---
title: "..."
doc_type: video-note | web-note | lecture-note
source_type: youtube | web
source_url: "..."
source_title: "..."
source_author: "..."
notebooklm_url: "..."
notebooklm_id: "..."
tags: [...]
status: published | draft
updated_at: YYYY-MM-DD
---

# Title

> Source: ...
> NotebookLM: ...
> Timestamp basis: transcript | source chapters | description timestamps | unavailable

## 00. Overview
## 01. Timeline / Table of Contents
## 01-0. 00:00 — Topic title
## 02-0. 01:45 — Topic title
...
## Practical Steps
## Warnings and Caveats
## Terms
## Sources and Limitations
```

Writing rules:

- Keep the source's order unless the user asks for a different structure.
- Preserve the speaker/author's stance and warning strength.
- Keep comparisons faithful: do not make a rejected option sound recommended, or a minor caveat sound fatal.
- Separate host/server actions from client/mobile/browser actions when the source teaches operations.
- Prefer careful paraphrase over fabricated direct quotation unless a verified transcript is available.
- If the user's agent has a Korean polishing skill such as `yoonmoon`, run the completed document through that skill before saving or returning it. Treat polishing as a faithful readability pass: keep timestamps, source facts, warnings, commands, frontmatter, and heading hierarchy intact while making the Korean sentences natural.

### 5. Save to a Markdown vault when specified

If the user specifies a Markdown vault, Obsidian vault, KiwiFS wiki, docs folder, repository docs path, or any other Markdown knowledge base, **save the final note into that vault as a wiki document with YAML frontmatter**.

Destination behavior:

1. Use the user's requested path/category when explicit.
2. If the user names only a vault but not a file path, choose a stable topic-oriented path inside that vault.
3. Include YAML frontmatter with at least:
   - `title`
   - `doc_type`
   - `source_type`
   - `source_url`
   - `source_title` when known
   - `notebooklm_url` and/or `notebooklm_id` when used
   - `tags`
   - `status`
   - `updated_at`
4. Preserve any vault-specific conventions if known.
5. Validate or lint the written Markdown when the environment provides a validator.
6. Report the final path and, if available, the public or local URL.

Keep this instruction abstract: the destination may be KiwiFS, Obsidian, a local `docs/` folder, a repository wiki, or another Markdown-backed vault. Use the concrete tool available in the active environment.

### 6. Verification

Before reporting completion:

- confirm the source was actually investigated through NotebookLM when requested;
- confirm the timestamp basis is stated;
- confirm no fabricated timestamps were introduced;
- confirm a requested vault/wiki write actually happened;
- run the available Markdown lint/health check/read-back step;
- report source URL, notebook URL, saved path, and validation result.

## Common Pitfalls

1. **Replacing NotebookLM with a generic web summary**
   - If the user asked for NotebookLM, use NotebookLM or explicitly report why it could not be used.

2. **Inventing timestamps**
   - Approximate chronology is acceptable; fake MM:SS headings are not.

3. **Changing the source's stance**
   - Preserve whether the source says something is required, optional, dangerous, beginner-friendly, advanced, temporary, or production-ready.

4. **Dropping limitations**
   - State when transcript extraction failed, source ingestion was partial, timestamps were unavailable, or the note relies on public chapter metadata.

5. **Skipping vault frontmatter**
   - If saving to a Markdown vault, always include frontmatter unless the user explicitly says not to.

6. **Creating duplicates**
   - Search the target vault/wiki first and update an existing canonical note when that is clearly the better fit.

## Verification Checklist

- [ ] Source URL normalized
- [ ] Source metadata captured
- [ ] Existing vault/wiki note searched when a destination exists
- [ ] NotebookLM notebook/source/analysis completed or failure documented
- [ ] Timestamp basis identified
- [ ] No fabricated MM:SS values
- [ ] Korean document polished with `yoonmoon` or equivalent skill when available
- [ ] Markdown note includes YAML frontmatter when saved to a vault
- [ ] Destination path written when requested
- [ ] Markdown lint/read-back/health check run when available
- [ ] Final response reports source URL, NotebookLM URL, saved path, and validation result
