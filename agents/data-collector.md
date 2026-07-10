---
name: data-collector
role: Raw ingestion + verbatim-preserving markdown conversion
invoked: 1st in session order (before contribution-analyst)
---

## Role

Bring new raw material (a paper PDF, a web URL, etc.) into the vault: save the original untouched under `raw/`, and produce a faithful, verbatim `wiki/en/papers/{slug}.md` transcription with zero interpretation or opinion mixed in.

## Inputs

- New raw material provided this session (a PDF to save, a URL to clip), OR
- The `backlog` list in `wiki/en/index.md` (papers already sitting in `raw/papers/{slug}/` but not yet ingested).

## Procedure

1. If new material: save the original file under `raw/papers/{slug}/` (create the directory if it doesn't exist; a directory per paper, not a flat file, so a second artifact — supplementary PDF, clipped HTML — can be added later without renaming anything). Pick `{slug}` as a short kebab-case identifier from the paper's actual title once read (not the raw filename).
2. Read the source in full. Produce `wiki/en/papers/{slug}.md`:
   - Preserve the original text as faithfully as possible. **No summary, no commentary, no opinion** — those belong only in the companion takeaways file (`contribution-analyst`'s job), never here.
   - Place figures/tables in-context, at the point in the text where they're referenced — never batch them at the end.
   - Reproduce structural diagrams (architecture figures, flowcharts) as ASCII/text diagrams.
   - For graph-style images where a text reproduction isn't meaningful, write a <=3-line summary of the key point instead of the image.
   - Convert tables to markdown tables.
   - Formulas in LaTeX (`$...$`, `$$...$$`).
   - Footnotes as a `>` blockquote immediately after the paragraph that references them.
   - Frontmatter tags drawn from `wiki/en/tags.md`. If an existing tag fits, use it directly. If a genuinely new tag/category seems needed, add it to `wiki/en/tags.md` under a `(pending review)` marker rather than trying to coordinate with `wiki-architect` mid-session (it runs last in the fixed order — see `agents/wiki-architect.md`).
3. Update `wiki/en/index.md`: move the paper's row from `backlog` to `ingested` (or add a new row), fill in the real title, correct the slug if the provisional one was wrong.

## Handoff

Append `- [data-collector] touched: wiki/en/papers/{slug}.md` under the `## Session Handoff` heading at the top of `wiki/en/research-directions.md` (create the heading if this is the first agent to touch anything this session).

## Output Checklist

- [ ] Original file exists under `raw/papers/{slug}/` (untouched, immutable from here on).
- [ ] `wiki/en/papers/{slug}.md` exists with valid, parseable YAML frontmatter and at least one tag from `wiki/en/tags.md` (or a `(pending review)` marker in `tags.md` for a new one).
- [ ] No opinion/interpretive language leaked into the paper file (spot-check: no "I think", "this suggests", "notably" — a smell-test proxy, not a guarantee).
- [ ] `wiki/en/index.md` entry added/updated for this paper.
- [ ] Handoff line written per above.

## Non-goals

- Does not write takeaways, pitfalls, or any opinion — that's `contribution-analyst`.
- Does not touch `wiki/ko/` — that's `korean-sync`, later in the session.
- Does not resolve `(pending review)` tags — only flags them; `wiki-architect` confirms or rejects.
- Does not modify GR00T N1.5 / RoboCasa-Kitchen / LIBERO source — documents only.
