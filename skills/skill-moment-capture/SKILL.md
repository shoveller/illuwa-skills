---
name: skill-moment-capture
description: "Use when a session reveals a reusable judgment, checklist, workflow, or failure-prevention rule that should become a skill or be added to an existing skill. Trigger: /skill-moment-capture"
allowed-tools: Bash(mkdir:*), Bash(grep:*), Bash(git:*), Read, Write, Edit, MultiEdit
---

# Skill Moment Capture

Use this skill to convert a useful session moment into reusable agent behavior without turning one-off notes into prompt clutter.

## Capture Triggers

Capture a moment when at least one condition is true:

- The same judgment is made twice in one session.
- The user asks for a method, essence, playbook, or reusable way of working.
- A failure mode appears that a future checklist could prevent.
- A project convention is discovered through code investigation.
- A tool sequence proves useful and repeatable.
- A refactor reveals a transferable boundary rule.

## Do Not Capture

- One-off implementation details.
- Secrets, credentials, private tokens, or machine-specific auth data.
- Temporary workarounds without a known trigger.
- Broad personality instructions that do not change concrete behavior.
- Tool behavior that should be implemented as a plugin instead of documentation.

## Decision Flow

1. Name the moment in one sentence.
2. Identify the future trigger phrase or situation.
3. Decide whether this updates an existing skill or creates a new one.
4. Extract commands, paths, safety rules, and verification steps.
5. Remove local trivia that will not help a future task.
6. Write the skill as operational instructions, not a retrospective.
7. Tell the user where the skill was written and whether restart is required.

## New Skill vs Existing Skill

Create a new skill when:

- The trigger is distinct from existing skills.
- The workflow has at least three reusable steps.
- The skill can stand alone without the session transcript.

Append to an existing skill when:

- The trigger already belongs to that skill.
- The new content is a rule, checklist item, or example.
- Creating a new skill would split one workflow across two places.

Prefer not to write a skill when:

- `AGENTS.md` passive context is a better fit.
- The user only needs a project note or memory.
- Native skill discovery would not help because same-session hot reload is required.

## Skill Shape

For project-local skills, prefer this path pattern:

```text
<project-root>/.agents/skills/<skill-name>/SKILL.md
```

For shareable `skills.sh` repositories, prefer this path pattern:

```text
skills/<skill-name>/SKILL.md
```

Use frontmatter even for project-local markdown skills:

```yaml
---
name: short-hyphenated-name
description: Use when ...
---
```

Then write:

- Purpose and trigger.
- Workflow.
- Rules and anti-patterns.
- Verification.
- Project-specific paths only when they are intentionally reusable.

## Session Note Template

```markdown
Moment: <what happened>
Trigger: <when future agents should use it>
General rule: <portable principle>
Project-specific rule: <optional path or command>
Verification: <how to know the skill worked>
Destination: new skill | append to <skill>
```

## Verification

- The skill answers when to use it.
- The workflow can be followed without reading the original session.
- The content contains no secrets.
- The file path matches the target skill loader's expected layout.
- If using native OpenCode skill discovery, the final answer says restart may be required.
