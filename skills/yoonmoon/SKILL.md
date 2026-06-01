---
name: yoonmoon
description: Use when a Korean text should be polished so it no longer reads like AI- or translation-generated prose while preserving meaning, facts, numbers, proper nouns, quotations, genre, register, and source intent. Ported from epoko77-ai/im-not-ai humanize-korean.
version: 1.0.0
author: Hermes Agent, adapted from epoko77-ai/im-not-ai
license: MIT
metadata:
  hermes:
    tags: [korean, writing, editing, humanize, ai-tell, translationese, yoonmoon]
    related_skills: []
---

# Yoonmoon — Korean AI-Tell Removal and Faithful Polishing

## Overview

`yoonmoon` is a skills.sh-compatible port of the `humanize-korean` Claude Code skill from <https://github.com/epoko77-ai/im-not-ai>. It helps an agent rewrite Korean text that sounds like ChatGPT, Claude, Gemini, machine translation, or post-edited translation into more natural Korean.

The governing rule is **meaning preservation before fluency**. The skill may change sentence rhythm, connective tissue, Korean particles, passive constructions, overused AI phrases, formatting, and register-specific awkwardness, but it must not add, delete, soften, intensify, or reorder the writer's claims.

Use the reference files as the single source of truth:

- `references/quick-rules.md` — fast S1/S2 pattern list and self-check checklist.
- `references/ai-tell-taxonomy.md` — full taxonomy of Korean AI-tell patterns.
- `references/rewriting-playbook.md` — practical rewrite recipes by category.
- `references/scholarship.md` — academic/source notes behind the taxonomy.
- `references/metrics.py` and `references/metrics_v2.py` — optional quantitative helpers from the upstream project.

## When to Use

Use this skill when the user asks for any of these Korean-writing tasks:

- "AI 티 없애줘", "ChatGPT 티 제거", "GPT 문체 고쳐줘";
- "사람이 쓴 것처럼 자연스럽게 윤문";
- Korean translationese cleanup, especially English/Japanese-style Korean;
- reducing over-mechanical headings, bullets, emoji, "첫째/둘째/셋째", or conclusion formulas;
- preserving content while only improving expression, rhythm, and Korean naturalness;
- second-pass polishing where the user says "이 문단만", "더 자연스럽게", "특정 카테고리만 다시".

Do not use this skill for:

- translation into or out of Korean;
- substantive rewriting that changes claims, structure, facts, or argument strength;
- copywriting that intentionally adds new persuasion, metaphors, examples, or brand voice;
- simple typo/spelling fixes where the user did not ask to remove AI-like prose;
- bypassing a detector by deception. Frame the task as faithful Korean editing, not as hiding authorship.

## Inputs to Identify

Before editing, infer or ask only when genuinely missing:

- source text and requested scope: whole text, section, paragraph, or selected category;
- genre/register: column, report, blog, official speech, academic, casual, product copy, etc.;
- desired intensity: conservative, default, active;
- whether formatting must be preserved exactly;
- whether direct quotations, legal wording, numbers, model names, product names, and citations appear.

Default assumptions when unspecified:

- mode: fast;
- intensity: default;
- genre: infer from the first 300 characters;
- formatting: preserve headings/lists only when they seem meaningful; otherwise reduce AI-like decoration;
- output: polished Korean text plus a short edit report.

## Non-Negotiable Invariants

1. **Meaning invariance** — facts, claims, stance, causality, scope, negation, modality, numbers, names, dates, units, and quotations remain identical in meaning.
2. **No invented content** — do not add examples, evidence, metaphors, claims, citations, or explanations that are not in the input.
3. **Genre and register preservation** — a report remains a report, a column remains a column, a formal text remains formal.
4. **Span-grounded editing** — every non-trivial change should correspond to a detected AI-tell or Korean-naturalness issue.
5. **Over-polish guard** — change rate above 30% requires caution; above 50% usually means stop, report risk, and ask for human review.
6. **Do-not-touch spans** — preserve proper nouns, product/model names, institution names, numbers, dates, units, direct quotes, legal/regulatory clauses, formulas, and standard abbreviations such as LLM, GPU, MCP, API.

## User Style Overrides

Apply these local preferences whenever they do not conflict with meaning preservation or genre:

0. **Treat a good Korean source as the style anchor.** If the user provides source text and says it is good, natural, final, or free of English/translation tone, preserve its macro-structure, rhetorical density, heading rhythm, emphasis, and key phrasing as much as possible. Do not flatten it into dry agent prose just because it is a technical document. The default edit is minimal: remove only the requested issues, convert endings if asked, and keep the source's confident Korean cadence.
1. **Use Korean technical-document plain style.** For manuals, lecture notes, runbooks, and technical docs, use declarative endings such as `~한다`, `~된다`, `~이다`; do not use polite present endings such as `~합니다`, `~됩니다`, `~입니다` unless the source genre requires polite speech. When converting `합니다` to `한다`, preserve the original sentence order and vocabulary unless there is a concrete awkwardness to fix.
2. **Preserve meaning while tightening into manual prose.** Keep facts, warnings, and emphasis unchanged, but remove filler only when it is actually filler. Prefer subject omission when the subject is obvious in Korean.
3. **Prefer active voice and direct predicates.** Convert needless passive or translationese to active wording when the actor is clear or can remain implicit: `설정됩니다` → `설정한다`, `사용됩니다` → `사용한다`, `확인됩니다` → `확인한다`.
4. **Minimize weak report-style formulas.** Avoid or reduce `~로 언급됩니다`, `~로 확인됩니다`, `~할 수 있습니다`, `~하는 것이 가능합니다`. Prefer direct forms: `~로 언급한다`, `~로 확인한다`, `~할 수 있다`, or stronger wording when the original meaning permits.
5. **Soften overly colloquial or harsh operational wording.** In guides, lecture notes, reports, and runbooks, replace casual verbs with calmer task-oriented verbs when the meaning permits: `죽는다` → `멈춘다`, `잡는다` → `설정한다` or `연결한다` depending on context.
6. **Unpack dense Sino-Korean compounds.** When a compressed or heavy 한자어 makes the sentence stiff, replace it with plain Korean without weakening the claim: `고권한`/`고권` → `높은 권한`, `권한 상승` → `권한이 높아짐` when appropriate.
7. **Compress translated English essay tone into dry technical-blog prose.** Avoid English-essay calques and explanatory filler such as `~라는 관점`, `문제의식`, `유용하다`, `실생활`, and `충분하지 않다`. Prefer short sentences, verbs over abstract nouns, minimal connectives, and direct wording. `핵심은 ~이다` is acceptable; extended essay-like framing is not.
8. **Use clear topic-first sentences.** Put the action or conclusion first when it saves the reader time, then separate conditions or caveats into the next sentence. Prefer `세션을 백그라운드로 분리한다. 세션을 끝내지 않는다.` over `세션을 완전히 끝내지 않고, 백그라운드로 분리합니다.`

Example:

```text
Input:
설치 후 tmux -V로 버전을 확인합니다. 강의 기준 버전은 3.5A로 언급됩니다.

Output:
설치 후 tmux -V로 버전을 확인한다. 강의 기준 버전은 3.5A로 언급한다.
```

## Modes

### Fast Mode — default

Use for normal requests and text up to roughly 5,000 Korean characters.

1. Load `references/quick-rules.md`.
2. Detect the strongest S1/S2 patterns in memory.
3. Rewrite locally, paragraph by paragraph.
4. Run the six-point self-check from `quick-rules.md`.
5. Return the polished text and a concise summary.

Fast mode should avoid multi-agent orchestration unless the host agent environment provides a cheap and reliable subagent mechanism. The upstream Claude skill used a `humanize-monolith` fast path; in this port, the current agent should perform that monolithic pass itself.

### Strict Mode — when requested or needed

Use strict mode when the user explicitly asks for precision, the text is long, previous output was rejected, or meaning preservation is risky.

Run the work as separate passes, even if done by one agent:

1. **Detection pass** — create a finding list with category ID, severity, span, reason, and suggested fix.
2. **Rewrite pass** — edit only finding-backed spans using `rewriting-playbook.md`.
3. **Fidelity audit** — compare original and rewrite for factual/stance drift.
4. **Naturalness review** — check remaining S1/S2 AI-tell patterns and over-polish.
5. **Revision loop** — at most three passes; if unresolved, report the difficult spans instead of forcing a rewrite.

If the runtime has parallel subagents, the fidelity audit and naturalness review can be assigned to separate reviewers. If not, perform them sequentially and label the limitations.

## Core Taxonomy Summary

Use the full taxonomy file for details. The common high-signal categories are:

| ID | Category | Typical signals | Preferred action |
|---|---|---|---|
| A | Translationese | `~를 통해`, `~에 대해`, `~에 있어서`, `~에 의해`, `가지고 있다`, `되어진다`, `그/그녀/그것` overuse | Restore Korean particles, active voice, zero pronouns, and natural clause order. |
| B | Excess English | repeated Korean + English parentheses, needless English terms | Keep standard abbreviations and proper nouns; reduce repeated or unnecessary English. |
| C | Structural AI patterns | mechanical `첫째/둘째/셋째`, repeated `X: Y` headings, bullets, emoji | Convert decorative structure into prose unless genre requires it. |
| D | Signature AI phrases | `결론적으로`, `시사하는 바가 크다`, `주목할 만하다`, hype adjectives | Delete or replace with concrete conclusions from the source text. |
| E | Rhythm | uniform sentence length, repeated endings, `~고 있다` overuse | Mix sentence lengths and Korean endings without changing register. |
| F | Over-modification | `~적`, `~성`, `~화`, duplicate adjectives, nominalization chains | Return to concrete verbs/adjectives or simpler noun phrases. |
| G | Hedging | `~할 수 있을 것으로 보인다`, excessive balance language | Strengthen only where the original meaning permits; keep uncertainty if real. |
| H | Connectives | sentence-initial `또한`, `따라서`, `즉`, `나아가` repeated | Remove many; let sentence order carry logic. |
| I | Bound nouns | `~것이다`, `~라는 점`, `~할 필요가 있다` | Replace with direct Korean predicates when safe. |
| J | Visual decoration | excessive bold, quotes, em dash, list styling | Reduce decoration while preserving useful structure. |

Severity convention:

- **S1 critical** — usually remove whenever found.
- **S2 strong** — tolerate isolated natural uses; remove repeated density.
- **S3 weak** — adjust only when it reinforces other patterns.

## Fast Workflow

1. **Save or isolate the input** if operating on files. Keep the original intact.
2. **Estimate genre/register** from the text, not from your preferred style.
3. **Scan for S1 first**: double passive, `가지고 있다`, formulaic conclusions, emoji/decorative headings, pronoun translationese, repeated connectives.
4. **Scan for user style overrides**: polite present endings in technical docs, passive/report-style formulas, passive verbs that can become active, overly colloquial operational verbs, compressed 한자어 such as `고권한`, English-essay calques such as `~라는 관점`, `문제의식`, `유용하다`, `실생활`, `충분하지 않다`, and sentences that hide the main action behind a condition or subordinate clause.
5. **Scan for S2 density**: `통해`, `에 대해`, `할 수 있다`, hedging, `것이다`, nominalization, uniform sentence lengths.
6. **Rewrite in this order**:
   1. D — signature phrases and hollow conclusions;
   2. A — translationese and Korean particle/voice repairs;
   3. I/G — bound nouns, unnecessary possibility, hedging;
   4. H/F/B — connectives, modifiers, English terms;
   5. C/J — structure and decoration;
   6. E — final rhythm pass.
7. **Protect do-not-touch spans** before and after editing.
8. **Self-check** against `quick-rules.md`.
9. **Report** what changed and any residual risk.

## Output Format

For short and medium text, return:

```markdown
완료. 변경률 약 X% / 등급 A|B|C|D / 자체검증 N/6 통과

## 윤문본
...

## 점검 요약
| 항목 | 결과 |
|---|---|
| 주요 탐지 | A-2, D-1, H-1 ... |
| 보존 확인 | 고유명사·수치·인용 보존 |
| 주요 변경 | "..." → "..." |
| 잔존 위험 | 없음 / ... |
```

When the user wants only the polished text, keep the report to one short line or omit it after the text.

When editing files, write the polished output to a new file unless the user explicitly asked to overwrite. Keep an original backup or rely on version control.

## Quality Grades

- **A** — S1 residual 0, S2 residual no more than 2–3 meaningful cases, change rate roughly 10–25%, no fidelity concern.
- **B** — S1 residual 0, S2 residual no more than 4, minor awkwardness remains, no fidelity concern.
- **C** — S1 residual 1–2 or self-check weakness; recommend strict mode or human review.
- **D** — S1 residual 3+, suspected meaning drift, or change rate above 50%; stop or ask before continuing.

## Handling Follow-Up Requests

| User request | Behavior |
|---|---|
| "이 문단만" | Treat only that paragraph as input; do not rework surrounding text. |
| "특정 카테고리만" | Re-scan and edit only the named category, e.g. translationese or connectives. |
| "더 자연스럽게" | Run strict review; avoid adding new voice or literary flair. |
| "원래보다 덜 바꿔" | Roll back lower-severity changes; keep only S1 and dense S2 fixes. |
| "격식 있게/블로그처럼" | Confirm whether this is a genre/register change; if yes, it is no longer pure yoonmoon. |

## Common Pitfalls

1. **Changing the author's argument.** Removing hedging can turn uncertainty into certainty. Strengthen only when the original already implies certainty.
2. **Making everything literary.** Human-sounding Korean is not automatically essayistic or poetic.
3. **Deleting useful structure.** Bullets, headings, and bold can be valid in manuals or reports; remove them only when they are decorative AI residue.
4. **Over-translating technical terms.** API, LLM, GPU, MCP, model names, product names, and standard domain terms often should remain as-is.
5. **Touching direct quotes.** Quoted text is evidence, not style material. Leave it intact unless the user explicitly asks to edit the quote.
6. **Assuming every `~의` is wrong.** The taxonomy targets double-particle forms like `~에서의`, `~에로의`, `~으로부터의`; simple `~의` is not automatically an error.
7. **Treating detector-bypass as the goal.** The goal is faithful Korean editing. Do not promise evasion of external AI detectors.
8. **Leaving operational text too rough.** In manuals and runbooks, avoid unnecessarily vivid verbs such as `죽는다` or `잡는다` when `멈춘다`, `설정한다`, or `연결한다` carries the same meaning more calmly.
9. **Keeping translated English essay framing.** For technical blogs, lecture notes, and runbooks, do not keep explanatory essay scaffolding such as `문제의식`, `~라는 관점`, `유용하다`, `실생활`, or `충분하지 않다`; compress it into short, direct technical prose.
10. **Burying the point in a long subordinate clause.** If a sentence starts with a caveat or condition and delays the main action, split it into topic-first sentences so the reader sees the intent immediately.

## Verification Checklist

- [ ] Original and revised text make the same claims with the same strength.
- [ ] Proper nouns, numbers, dates, units, citations, and direct quotes are preserved.
- [ ] Genre and register match the input unless the user asked otherwise.
- [ ] S1 patterns were removed or explicitly justified as retained.
- [ ] Repeated S2 density was reduced without over-editing.
- [ ] User style overrides were checked when applicable: technical-document plain style (`~한다`, `~된다`, `~이다`), concise subject-light manual prose, passive/report-style formula reduction, passive→active, overly colloquial operational verbs softened, dense Sino-Korean compounds unpacked, translated English essay tone compressed into dry technical-blog prose, and buried main actions rewritten as clear topic-first sentences without meaning drift.
- [ ] No new examples, metaphors, facts, or causal claims were added.
- [ ] Change rate feels below 30%; if not, risk is reported.
- [ ] The user receives either the polished text or a file path plus a concise report.

## Source and License Note

This skill is a portable adaptation of `humanize-korean` from `epoko77-ai/im-not-ai` at <https://github.com/epoko77-ai/im-not-ai>. The upstream repository is MIT licensed. The port keeps the operational ideas and reference materials while removing Claude-Code-specific assumptions where possible.
