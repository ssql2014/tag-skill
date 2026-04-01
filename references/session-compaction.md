# Session Compaction

Use this reference when the goal is to compress a long tagged conversation into
a much shorter working memory summary.

## Principle

`k` blocks are candidates for survival, not guaranteed verbatim output.

The compactor should produce a current snapshot, not an archive dump.

## Best Compression Strategy

1. Gather all `k` blocks since the last compact or checkpoint.
2. Group them by topic.
3. Within each topic, keep the newest still-valid state and discard superseded
   earlier statements.
4. Rewrite the survivors into a short session snapshot.

## Preferred Output Shape

A good compacted session summary usually has only these sections:

1. Standing conventions
2. Current durable state
3. Open blockers or risks
4. Durable anchors
5. Next pending actions

## What Usually Survives

- operating rules the agent must keep following
- durable decisions
- root causes
- current owner/task state
- still-open blockers
- file paths, ids, hosts, issue numbers, commit ids, session names

## What Usually Gets Dropped

- intermediate probes
- repeated confirmations
- retries
- obsolete plans
- completed micro-actions
- draft thoughts that never became the chosen path

## Compression Heuristics

- Prefer latest-wins over timeline-preserving.
- Prefer state over narration.
- Prefer one merged statement over several near-duplicate `k` blocks.
- Prefer anchors over prose.
- If a `k` block is recoverable cheaply from code, logs, or the current screen,
  it often did not need to be `k` in the first place.

## Practical Rule

Aim for sparse `k` tagging during the live conversation so compaction becomes
mechanical later.

If too many lines look like `k`, the threshold is wrong. Tighten the live
tagging, not the final compactor.
