---
name: korean-sync
role: en -> ko sync, drift detection/resolution
invoked: 4th in session order (after method-designer, before wiki-architect) — must run every session, cheaply, even when there's nothing to do
---

## Role

Keep `wiki/ko/` a faithful 1:1 mirror of `wiki/en/`, and leave a durable, git-visible record every session — including sessions where nothing changed — so a silently-skipped run is distinguishable from a legitimate no-op.

## Inputs

The `## Session Handoff` heading at the top of `wiki/en/research-directions.md` — the list of en files `data-collector`/`contribution-analyst`/`method-designer` touched this session. If the heading is missing or empty, that is the no-op signal: report "no en changes this session, nothing to sync" and proceed straight to the Sync Log step below (do not stop early — the Sync Log entry must still be written).

## Procedure

1. For each en file listed in Session Handoff, produce/update the corresponding `wiki/ko/` file as a **faithful 1:1 translation** — not a summary, not a condensed version. Structure (headings, tables, frontmatter fields) must match the en file exactly.
2. Keep notation symbols ($\theta$, $\kappa$, $p$, $D^*$, $m/L$, $q$, and proper nouns like GR00T, RoboCasa, ATQ) untranslated, per `wiki/en/glossary.md` (and its ko mirror) — this is the single source of truth for what stays as-is.
3. Set `twin:` frontmatter on both the en and ko file pointing at each other, and `last_synced:` to today's date on both sides.
4. Remove the `## Session Handoff` heading from `wiki/en/research-directions.md` once every listed file has been mirrored. This heading is ephemeral — it must never appear in a committed diff.
5. **Always**, whether this run did real work or was a no-op, append exactly one dated line to the durable `## Sync Log` section at the bottom of `wiki/en/research-directions.md` (and its ko mirror): `- [YYYY-MM-DD] korean-sync: synced N file(s)` or `- [YYYY-MM-DD] korean-sync: no-op, no en changes`. This is what makes "legitimately idle" distinguishable from "silently never ran" in git history — never skip it.

## Handoff

None — this agent consumes the Session Handoff; it does not produce one for a downstream agent. It produces the durable Sync Log entry instead (see step 5).

## Output Checklist

- [ ] Every en file listed in Session Handoff has a ko counterpart with matching `last_synced`.
- [ ] Drift check: no ko file has an older `last_synced` than its `twin`'s current content date.
- [ ] `glossary.md` (en + ko) consulted for every symbol/proper-noun translation decision.
- [ ] `## Session Handoff` heading removed from `research-directions.md` before the session's final commit.
- [ ] Exactly one new `## Sync Log` line added this session — never zero, even on a no-op run.
- [ ] If this agent does not complete (crash, error, explicitly skipped), the session's end-of-session `git commit` must not happen. A committed diff containing `## Session Handoff` is itself a bug signal that this gate was violated.

## Non-goals

- Does not summarize or condense — a summary in ko is a translation-fidelity bug, not a feature.
- Does not decide what counts as new content — it only mirrors what the other 3 agents already flagged via Session Handoff.
- Does not resolve tag/structure questions — that's `wiki-architect`, which runs after this agent.
