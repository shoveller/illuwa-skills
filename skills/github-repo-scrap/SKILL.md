---
name: github-repo-scrap
description: Use when asked to scrap, summarize, or record one or more GitHub repository URLs into a Markdown/KiwiFS wiki for future reuse. Trigger with "레포 스크랩", "깃허브 스크랩", "repo scrap", or bare "스크랩" when GitHub repository URLs are present.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [github, repository, scrap, wiki, kiwifs, open-source]
    related_skills: [kiwifs, github-oss-candidate-vetting]
---

# GitHub Repo Scrap

## Overview

`github-repo-scrap` turns one or more GitHub repository URLs into short, reusable wiki notes. Use it when the user wants to remember why a repository exists, what it does, and where it may be useful later.

This is a **lightweight discovery and recording workflow**, not a full due-diligence review. The output should help a future agent or human quickly decide whether the repository is worth revisiting.

Core questions:

1. What problem did this repository appear to solve?
2. What does it do?
3. Where might it be useful later?
4. What caveats, unknowns, or maintenance risks should be remembered?

## When to Use

Use this skill when the user says things like:

- `레포 스크랩`
- `깃허브 스크랩`
- `repo scrap`
- `GitHub repo scrap`
- `이 레포 위키에 기록해줘`
- `다음에 쓸 수 있게 이 GitHub 저장소 소개 남겨줘`
- `스크랩 https://github.com/owner/repo`

Use it for:

- one GitHub repository URL;
- multiple GitHub repository URLs;
- pasted lists of GitHub repositories;
- `owner/repo` shorthand when it clearly means GitHub;
- GitHub issue, PR, release, or tree URLs when they can be reduced to the repository.

Do not use it for:

- non-GitHub article/video/document scraping;
- full security review;
- PR/code review;
- local installation or debugging;
- exhaustive alternative comparison;
- private repositories unless the user has explicitly provided authorized access and asks to use it.

## Trigger Rule

Prefer specific trigger phrases:

```text
레포 스크랩
깃허브 스크랩
repo scrap
```

The bare word `스크랩` is accepted **only when a GitHub repository URL is present**.

Examples:

```text
스크랩 https://github.com/modelcontextprotocol/servers
```

Use this skill.

```text
이 블로그 스크랩해줘
```

Do not use this skill; route to a general web/article scraping workflow.

## Input Normalization

Accept these input forms:

```text
https://github.com/owner/repo
```

```text
- https://github.com/owner/repo-a
- https://github.com/owner/repo-b
```

```text
owner/repo
```

Normalize `owner/repo` to:

```text
https://github.com/owner/repo
```

Reduce deep GitHub URLs to the repository when possible:

| Input shape | Repository |
|---|---|
| `https://github.com/owner/repo/issues/123` | `owner/repo` |
| `https://github.com/owner/repo/pull/123` | `owner/repo` |
| `https://github.com/owner/repo/releases/tag/v1.0.0` | `owner/repo` |
| `https://github.com/owner/repo/tree/main/path` | `owner/repo` |
| `https://github.com/owner/repo/blob/main/file.md` | `owner/repo` |

Ignore URLs that are not repository-like unless the user clarifies.

## Research Procedure

For each repository:

1. Normalize the repository URL and identify `owner`, `repo`, and canonical URL.
2. Search the target wiki or Markdown vault before writing:
   - exact GitHub URL;
   - `owner/repo`;
   - repository name;
   - package or project name if obvious.
3. Inspect lightweight public evidence:
   - GitHub repository description;
   - README;
   - docs directory or homepage if clearly linked;
   - package metadata if visible;
   - license;
   - latest release/tag or visible activity when relevant.
4. Identify the problem framing:
   - What pain, missing capability, repeated workflow, or technical gap does it address?
   - Who appears to need it?
5. Identify what it does:
   - main functions;
   - interface shape: CLI, library, server, MCP server, web app, GitHub Action, SDK, Docker image, etc.;
   - major integrations or runtime requirements if obvious.
6. Identify future usefulness:
   - where it might fit the user's stack, research, automation, wiki, MCP, Cloudflare, agent, or dev workflow;
   - what follow-up evaluation would be needed before adoption.
7. Record caveats:
   - unclear README claims;
   - abandoned or low-activity project;
   - license uncertainty;
   - hosted-service/API-key dependency;
   - security/auth implications;
   - missing tests/docs;
   - anything not yet verified.
8. Write or update the wiki note.
9. Run the available Markdown lint/read-back/health check for the destination.
10. Report the changed path and URL.

## Evidence Rules

Keep the note source-grounded.

- Prefer README/docs/repo metadata over guesses.
- Do not infer production readiness from stars alone.
- Do not treat README marketing claims as verified behavior.
- If implementation evidence was not inspected, say the note is README/docs-based.
- If the user asks whether the repository can satisfy a concrete workflow, escalate to a feasibility investigation rather than writing only a light scrap note.
- Never copy secrets, tokens, cookies, private config, or credentials into the wiki.

## Destination Rules

When the target is KiwiFS or another wiki and no path is specified, prefer:

```text
30 Wiki/github-repositories/<owner>/<repo>.md
```

Example:

```text
30 Wiki/github-repositories/modelcontextprotocol/servers.md
```

If the user's wiki already has a more specific canonical area, prefer updating that area over creating a duplicate. Examples:

- Cloudflare-related repository → existing Cloudflare category if one clearly exists.
- MCP server/library → existing MCP category if one clearly exists.
- Project-specific dependency → that project's existing notes area.

When multiple repositories share a theme, optionally update or create an index page such as:

```text
30 Wiki/github-repositories/README.md
```

Add compact index entries:

```markdown
- [[30 Wiki/github-repositories/owner/repo.md|owner/repo]] — one-line description.
```

## Page Template

Use Korean by default unless the user asks otherwise.

```markdown
---
title: "owner/repo"
doc_type: github-repo-note
source_type: github
source_url: "https://github.com/owner/repo"
owner: "owner"
repo: "repo"
status: active
tags:
  - github
  - repository
  - scrap
updated_at: YYYY-MM-DD
---

# owner/repo

## 한 줄 소개

이 레포가 무엇인지 한 문장으로 설명한다.

## 등장 배경 / 해결하려는 문제

이 프로젝트가 어떤 불편, 결핍, 반복 작업, 기술적 문제를 해결하려고 등장했는지 기록한다.

## 무엇을 하는가

- 핵심 기능 1
- 핵심 기능 2
- 핵심 기능 3

## 활용 가능성

다음 작업에서 어디에 쓸 수 있을지 짧게 기록한다.

예상 활용:

- 활용처 1
- 활용처 2
- 활용처 3

## 작동 방식 / 구성 요소

README나 문서에서 확인되는 범위 안에서만 기록한다. 모르면 추정하지 않는다.

## 설치·운영 형태

CLI, 라이브러리, 서버, Docker, SaaS 연동, GitHub Action 등 운영 형태를 간단히 기록한다.

## 주의점 / 미확인 사항

- 라이선스, 유지보수 상태, 인증/비용/보안, 문서 부족 등을 기록한다.
- 확인하지 못한 부분은 `미확인`으로 남긴다.

## 원문 / 근거

- GitHub: https://github.com/owner/repo
- README: https://github.com/owner/repo#readme
```

## Writing Style

- Use concise Korean technical prose.
- Keep one repository note short: usually 500–1,200 Korean characters.
- Use the order: problem → behavior → future usefulness → caveats.
- Avoid marketing language such as “강력한”, “혁신적인”, “완벽한” unless quoting a source.
- Prefer concrete verbs: `제공한다`, `연결한다`, `생성한다`, `자동화한다`, `대체한다`.
- Label uncertainty explicitly: `README 기준`, `문서상`, `미확인`, `추가 검증 필요`.
- Do not add adoption recommendations unless the evidence supports them.

## Multi-Repository Handling

When multiple repositories are supplied:

1. Process every normalized repository.
2. Create or update one page per repository.
3. Keep each note independent enough to be reused later.
4. If the list has a shared theme, add a short index or collection note.
5. In the final response, group results in a table:

```markdown
| Repo | Wiki path | Result |
|---|---|---|
| owner/repo | 30 Wiki/github-repositories/owner/repo.md | created/updated |
```

## Escalation to Deeper Review

Escalate beyond a light scrap note when the user asks:

- “이걸 우리 환경에 쓸 수 있나?”
- “Cloudflare에 배포 가능한가?”
- “MCP 서버로 쓸 수 있나?”
- “보안상 괜찮나?”
- “A와 B 중 무엇을 써야 하나?”
- “실제로 설치해서 확인해줘”

In those cases, inspect implementation evidence, configuration paths, runtime assumptions, and actual commands. Record the deeper result as a feasibility or evaluation note, not just a repository scrap.

## Verification Checklist

Before reporting completion:

- [ ] Every supplied repository URL was normalized.
- [ ] Existing wiki/vault notes were searched before creating a new page.
- [ ] README/docs/repo metadata were inspected.
- [ ] Each note answers the problem the repo addresses.
- [ ] Each note describes what the repo does.
- [ ] Each note includes future usefulness or possible follow-up use.
- [ ] Each note includes caveats or unknowns.
- [ ] Claims are grounded in visible sources.
- [ ] No secrets or private credentials were written.
- [ ] Markdown lint/read-back/health check ran when available.
- [ ] Final response includes changed paths and URLs when available.
