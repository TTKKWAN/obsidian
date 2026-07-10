# Wiki: VLA Temporal-Action-Quantization Research Vault

This file is the schema: it tells you (the LLM) how this vault is organized and how to maintain it every session. Read it before touching anything in `raw/`, `wiki/`, or `agents/`.

## Topic

Vision-Language-Action (VLA) models, focused specifically on **temporal action quantization** — the current project is a **Success-Grounded Quantization Signal Router**: a per-chunk module that decides, right before executing an action chunk, whether it's safe to dispatch to a coarse (temporally-compressed) vs. fine decoder, trained on labels grounded in actual mixed-quant rollout success rather than a reconstruction-fit proxy. It extends ATQ (Adaptive Temporal Quantization), the reference paper, rather than reproducing it. See `raw/proposal.md` for the canonical spec and `wiki/en/research-directions.md` for the evolving research direction.

## Session workflow (mandatory, every session)

- **Start of session:** `git pull`.
- **End of session:** summarize the diff to the user, then `git add -A && git commit -m "..." && git push`.
- This rule is self-enforcing — run it regardless of what else the session was asked to do.
- **Commit gate:** the end-of-session commit only happens if `agents/korean-sync.md`'s agent completed this session (ran to completion, including its no-op path — see below). If it crashed, errored, or was skipped, fix or re-run it first; do not commit. A committed diff containing a `## Session Handoff` heading in `wiki/en/research-directions.md` is itself a bug signal that this gate was violated.

## Directory structure

```
raw/
  proposal.md              Immutable. The project's own canonical spec — never duplicated
                            verbatim into wiki/, only linked to.
  papers/
    {slug}/
      original.pdf          Immutable source, one directory per paper (not a flat {slug}.pdf) so a
                            second artifact — supplementary PDF, clipped HTML — can be added later
                            without renaming anything. Mirrors the raw/assets/ precedent.
  assets/                   Images extracted from clipped web sources.

wiki/
  en/                       AI-native brain — written/updated first, every session.
    index.md                Catalog: papers (ingested/backlog), methods, research directions, links.
    tags.md                 Tag taxonomy — single source of truth. New tags added here before use.
    research-directions.md  Append-only dated log + in-place Open Questions + Session Handoff +
                            durable Sync Log (see Korean twin wiki rules below).
    glossary.md              Notation glossary (θ, κ, p, D*, m/L, q, ...) — the shared source of
                            truth for which symbols stay untranslated in wiki/ko/.
    papers/{slug}.md         Verbatim-preserving transcription. No opinion/commentary — ever.
    takeaways/{slug}-takeaways.md   Takeaways + pitfalls. Opinion lives here, nowhere else.
    methods/                 4 fixed files (see Method files below) — no ad hoc additions.
  ko/                       Human-native brain — 1:1 structural mirror of everything above,
                            produced in the same session as the en change it mirrors.

agents/                     5 fixed, reusable agent definitions (see Agent roster below).
CLAUDE.md                   This file.
```

## Raw ingestion & conversion rules

When bringing a new paper/URL into the vault (`agents/data-collector.md`):

- Preserve the original text as faithfully as possible in `wiki/en/papers/{slug}.md` — **no summary, no commentary, no opinion**; those go only in `wiki/en/takeaways/{slug}-takeaways.md`.
- Figures/tables go in-context, at the point in the text where referenced — never batched at the end.
- Structural diagrams (architecture figures, flowcharts) get reproduced as ASCII/text diagrams.
- Graph-style images that don't reproduce meaningfully as text get a <=3-line summary of the key point instead.
- Tables become markdown tables. Formulas are LaTeX (`$...$`, `$$...$$`). Footnotes become a `>` blockquote immediately after the referencing paragraph.
- Frontmatter tags are drawn from `wiki/en/tags.md`. A genuinely new tag gets added there under a `(pending review)` marker (see Agent roster — `data-collector` cannot synchronously coordinate with `wiki-architect`, which runs last).

## Takeaway/pitfall extraction rules

`agents/contribution-analyst.md`: each ingested paper gets `wiki/en/takeaways/{slug}-takeaways.md` — Takeaways usable in this project's research (each with a 1-line "why relevant"), and Pitfalls (paper-stated or judged). When takeaways change the research direction or open a new question, append a dated entry to `wiki/en/research-directions.md`'s Dated Log (append-only — never edit a past entry) and update the Open Questions section in place if affected.

## Method files

4 fixed files under `wiki/en/methods/`: `action-quantization.md`, `dataset-labeling.md`, `module-training.md`, `label-module-architecture.md`. Each follows: **Current Approach / Alternatives Considered / Open Sub-problems / Experiment Log (dated, chronological) / Decision Status** (`exploring` | `decided` | `implemented`).

Two standing open sub-problems must always be kept current (`agents/method-designer.md`):
1. **Statistical reliability of single-rollout success estimates** — lives in `action-quantization.md`.
2. **Generalization of quantization-safety labels to nearby state-space regions** (credit assignment / compositionality) — lives in `label-module-architecture.md`.

## Extend-don't-modify boundary

GR00T N1.5, RoboCasa-Kitchen, and LIBERO are referenced and documented only. No agent or workflow in this repo edits those codebases — decisions/experiments about them are recorded here as documentation; actual code changes happen in those projects' own repos.

## Korean twin wiki rules

Everything under `wiki/en/` gets mirrored 1:1 into `wiki/ko/` as a **faithful translation, not a summary** (`agents/korean-sync.md`). Every md file's frontmatter carries `twin: <relative-path>` linking en<->ko and `last_synced: <date>`. Notation symbols and proper nouns (θ, κ, p, D\*, m/L, q, GR00T, RoboCasa, ATQ, ...) stay untranslated per `wiki/en/glossary.md` — the shared source of truth `korean-sync` must consult before translating anything.

**Session Handoff mechanism:** `data-collector`/`contribution-analyst`/`method-designer` append one line each (if they touched anything) to a `## Session Handoff` heading at the top of `wiki/en/research-directions.md`, format `- [agent-name] touched: <file path>`. **These three agents must run strictly sequentially, never in parallel** — they share this one mutable heading, and concurrent read-modify-write would silently lose a line. `korean-sync` consumes the heading, mirrors every listed file, then deletes the heading — it must never survive into a commit.

**Fast-path clause:** if no `wiki/en/*` file changed this session, `korean-sync` is a no-op and must say so plainly — the mandatory-every-session rule exists to prevent drift, not to force translation work when there's nothing to translate.

**Durable Sync Log:** distinct from the ephemeral Session Handoff heading, `korean-sync` always appends one dated line to a durable `## Sync Log` section at the bottom of `research-directions.md` (both en and ko), every run including no-op ones — `synced N file(s)` or `no-op, no en changes`. This is what makes a legitimately-idle run distinguishable from a silently-skipped one in git history; a gap in the dated sequence is the detectable signature of a skipped session.

## Tag taxonomy governance

New tags must be added to `wiki/en/tags.md` (mirrored to `wiki/ko/tags.md`) before use — no ad hoc tags in individual files. `data-collector` may add a tag under `(pending review)`; only `wiki-architect` confirms or rejects it, at end of session.

## Agent roster

5 fixed, reusable agents under `agents/`, invoked in this order every session:

1. `agents/data-collector.md` — raw ingestion + verbatim conversion.
2. `agents/contribution-analyst.md` — takeaway/pitfall extraction, research-directions.md updates.
3. `agents/method-designer.md` — updates the 4 method files, manages Decision Status.
4. `agents/korean-sync.md` — en->ko sync, drift detection, durable Sync Log entry. **Must run every session** (cheaply, via its no-op fast path, when nothing changed) — see the commit gate above.
5. `agents/wiki-architect.md` — structure/tag/index consistency, resolves `(pending review)` tags, decides if new nodes are needed. Runs last.

Agents 1-3 must run strictly sequentially, never in parallel (see Session Handoff mechanism above).

## Structural change gate

After this initial bootstrap, only `agents/wiki-architect.md`'s agent may add/rename top-level directories, fixed method files, or tag categories. Everyone else works within the existing structure.
