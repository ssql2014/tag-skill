---
name: tag-skill
description: >
  Use when the user asks to tag replies, annotate blocks for later memory
  extraction, classify content as keep or discard, enforce a compact response
  envelope with start/end IDs, or create a grep-friendly in-chat checkpoint via
  `/tag` or an explicit memory tag request. Works for Codex, Claude, or similar
  chat agents.
---

# Tag Skill

Use this skill when the user wants structured tags in the conversation.

This skill is agent-agnostic. It is intended to work for Codex, Claude, or
other terminal/chat agents that can follow a compact reply format.

This skill supports two modes:

1. `checkpoint` mode for a one-shot in-chat memory tag
2. `envelope` mode for tagging normal replies block by block

Pick the mode that matches the user's request. Do not mix both in the same reply
unless the user explicitly asks for both.

## Mode Selection

Use `checkpoint` mode when the user says things like:

- `/tag`
- `tag this`
- `checkpoint this`
- `make a memory entry`
- `summarize this for later grep`

Use `envelope` mode when the user says things like:

- `tag replies`
- `annotate blocks`
- `mark keep vs discard`
- `use start/end ids`
- `format future replies for memory extraction`

If the thread already established a tagging convention, keep using it until the
user changes or stops it.

If the current agent platform has its own native wrapper syntax for skills,
adapt only the invocation mechanics. Keep the tag semantics and output format
the same.

## Checkpoint Mode

Use checkpoint mode for a single compact tag block that summarizes the work
since the last explicit tag or the currently relevant slice of work.

Reply with the tag block directly. Do not add extra explanation before or after
it.

Every checkpoint block must start with:

```text
@@TAG@@
```

Use exactly this shape:

```text
@@TAG@@
slug: <short-hyphen-slug>
tags: <comma-separated-tags>
status: <done|partial|blocked|verified|followup>
summary: <1-3 short sentences>
anchors: <anchor1>; <anchor2>; <anchor3>
```

Checkpoint rules:

- Keep it compact.
- Prefer durable nouns and workflow names over prose.
- Reuse existing tags when possible.
- Include only high-signal anchors such as file paths, hostnames, user ids,
  commit ids, ticket ids, or exact error strings.
- Do not dump the full transcript.

## Envelope Mode

Use envelope mode when the user wants normal assistant replies tagged block by
block.

Wrap each tagged reply in a start/end envelope:

```text
[start <id>]
[B1][<category>][<k|d>] content
[B2][<category>][<k|d>] content
[End <id>]
```

Example:

```text
[start 14]
[B1][finding][k] SSH to nuc is restored and now routes over utun7.
[B2][action][d] I re-ran a transient probe to confirm the fix.
[End 14]
```

Envelope rules:

- Use a matching `[start <id>] ... [End <id>]` pair for every tagged reply.
- Keep the same format consistently once the convention starts.
- Put each meaningful unit in its own block.
- If the response is naturally short, one block is enough.
- Continue the established reply id sequence when the thread already has one.
- If no prior id exists, start from a small integer and increment once per
  assistant reply.
- Treat `d` as the default. A tagged reply may contain zero `k` blocks.
- Prefer at most 0 to 2 `k` blocks per reply unless the user explicitly asks
  for a heavier memory capture.

## Tag Meanings

- `start <id>`: start of the reply envelope
- `End <id>`: end of the same reply envelope
- `B<n>`: block number inside the current reply
- `tool <id>.<n>`: optional tool-progress block identifier
- `<category>`: short content class
- `k`: keep for durable memory
- `d`: discard for temporary process detail

## Tool Update Format

When tool-call progress should also be tagged, use:

```text
[tool 14.1][action][d] Inspecting the current route to 100.119.126.37.
```

Guidelines:

- Use `tool <reply-id>.<n>` for tool-related progress inside a tagged reply.
- Keep tool blocks brief.
- Most tool blocks should be `d` unless they capture a durable command, root
  cause, or reusable workflow.

## Retention Rule

Interpret `k` and `d` strictly as memory-retention value, not urgency.

- Use `k` only for information that should survive the next session compaction.
- Use `d` for transient probes, intermediate retries, exploratory dead ends,
  and one-off operational noise.

### `k` Threshold

Use `k` only when all of these are true:

1. The fact will still matter after the next compact.
2. It would be expensive or risky to reconstruct later.
3. It is not likely to be superseded within a few turns.
4. It can be stated as a short stable delta, rule, anchor, or decision.

If any of these fail, use `d`.

Rule of thumb:

- `k` means "carry across compaction"
- `d` means "useful for now, safe to drop later"

Category defaults:

- `action`: usually `d`
- `result`: usually `d`
- `finding`: `k` only if it is a durable state or root cause
- `decision`: `k` only if the decision remains operative
- `meta`: `k` only for a standing protocol or formatting rule
- `risk`: `k` only for an ongoing risk that future turns must remember

Latest durable state wins. If a later block supersedes an earlier `k`, the
earlier one should be treated as discardable during compaction.

## Recommended Categories

Prefer this compact set:

- `meta`
- `finding`
- `result`
- `action`
- `hypothesis`
- `decision`
- `risk`

If none fit perfectly, choose the closest short label instead of inventing a
large taxonomy.

## Style Rules

- Keep tags compact.
- Do not add framing fluff around tagged output.
- If the user asks for shorter notation, preserve the semantics and shorten only
  the labels.
- The latest explicit rule from the user is authoritative for the current
  thread.
- If in doubt between `k` and `d`, choose `d`.

## Compaction Use

The best later compaction flow is not "keep every `k` line verbatim". Instead:

1. Collect candidate `k` blocks.
2. Drop any `k` block superseded by a later block on the same topic.
3. Merge the remainder into a compact state snapshot.
4. Keep only:
   - standing conventions
   - current world state
   - open blockers and live risks
   - durable anchors such as paths, ids, hosts, commits
   - immediate next actions that are still pending

For a fuller recipe, see `references/session-compaction.md`.

## When Not To Use

- Do not add tags unless the user asked for tagging, memory labels, structured
  references, or a previously established convention requires them.
- If the user explicitly asks to stop tagging, stop immediately.
