---
name: tag
description: >
  Use when the user says `/tag`, asks to tag or checkpoint the current work,
  or wants a grep-friendly memory entry summarizing the conversation since the
  last tag. Output only a compact in-chat tag block.
---

# Tag Memory

Use this skill when the user wants a lightweight checkpoint in the conversation,
not a file write and not a long report.

The goal is:
- summarize work since the last explicit tag in the current conversation
- assign a small set of stable tags
- record hard anchors for later lookup
- emit a grep-friendly block with a fixed token

## Output Rules

Keep the tag compact.

- `slug`: short hyphen-case identifier
- `tags`: 3 to 8 concise tags
- `status`: one of `done`, `partial`, `blocked`, `verified`, `followup`
- `summary`: 1 to 3 short sentences
- `anchors`: only high-signal identifiers
  - file paths
  - hostnames
  - user ids
  - commit ids
  - ticket ids
  - error strings

Do not dump a full session transcript.

## Grep Token And Format

Every tag block must start with:

```text
@@TAG@@
```

This token is reserved for grep and should not be changed.

Use exactly this shape:

```text
@@TAG@@
slug: <short-hyphen-slug>
tags: <comma-separated-tags>
status: <done|partial|blocked|verified|followup>
summary: <1-3 short sentences>
anchors: <anchor1>; <anchor2>; <anchor3>
```

## Response Style

Reply with the tag block directly.

Do not:
- write files
- mention scripts
- add extra explanation before the block
- turn it into a long report

## Tag Selection Heuristics

- prefer durable nouns and workflow names over prose
- reuse existing tags when possible
- include the main system and outcome
- include one blocker tag if unresolved

Good examples:
- `dingtalk,leave-report,email,alice`
- `openclaw,imap,monitor,mac02`
- `claude-migration,codex,fallbacks`

Bad examples:
- `stuff,we-did,a-lot`
- full sentences
- ephemeral wording that will not repeat
