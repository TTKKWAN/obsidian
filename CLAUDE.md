# Wiki: VLA Models — Temporal Action Quantization

This is a research deep-dive wiki following the pattern described in [PATTERN.md](PATTERN.md). This file is the schema: it tells you (the LLM) how this specific wiki is organized and how to maintain it. Read it before ingesting a source or answering a query.

## Topic

Vision-Language-Action (VLA) models, with a focus on **temporal action quantization** — how continuous action sequences get discretized into tokens for autoregressive prediction (action tokenizers, codebooks, FSQ/VQ-VAE variants, chunking/horizon choices, and how these choices trade off against diffusion/flow-matching action heads).

## Directory structure

```
raw/                  Immutable source materials. Never edit these.
  assets/             Downloaded images from clipped articles/papers (see Image handling)
wiki/
  index.md            Catalog of every wiki page — read this first when answering a query
  log.md               Append-only chronological record of ingests/queries/lints
  sources/            One summary page per ingested source (paper, article, post)
  entities/            Pages for concrete things: specific models, papers, orgs/labs, datasets, benchmarks
  concepts/            Pages for ideas/techniques that span multiple sources: e.g. action tokenization,
                       FSQ vs VQ-VAE, chunking horizon, autoregressive vs diffusion action heads
  synthesis/           Standing thesis, comparison tables, open questions — the evolving "so what"
CLAUDE.md              This file — the schema
PATTERN.md             The generic LLM-wiki idea doc this wiki instantiates (reference only, not maintained)
```

## Source formats

- **Papers**: drop the PDF as-is into `raw/` — no conversion needed, read it directly. Name it `<arxiv-id>-<short-slug>.pdf` (e.g. `2410.12345-fast-tokenizer.pdf`) so it's greppable and traceable back to arXiv.
- **Articles/blog posts**: clip with Obsidian Web Clipper — lands as markdown in `raw/` with images in `raw/assets/`.
- Raw sources are never edited or renamed after ingest — treat them as immutable.

## Page conventions

- Every wiki page gets YAML frontmatter:
  ```yaml
  ---
  type: source | entity | concept | synthesis
  title: <human-readable title>
  date: <YYYY-MM-DD ingested or last updated>
  tags: [tag1, tag2]
  sources: [<source page links this draws from>]
  ---
  ```
- File naming: kebab-case, descriptive (`fast-tokenizer.md`, `openvla.md`, `action-chunking.md`).
- Use `[[wikilink]]`-style relative markdown links (`[FAST](../concepts/fast-tokenizer.md)`) so Obsidian's graph view stays useful.
- Every source page must link out to at least one concept or entity page, and every concept/entity page should link back to the sources that informed it. Avoid orphan pages — check `index.md` periodically for anything with no inbound links.
- Keep concept pages synthesis-oriented, not per-source notes: when a new source touches an existing concept, *update* that concept page (add nuance, flag contradictions, revise claims) rather than duplicating it in the source summary.

## Workflows

### Ingest (default: one source at a time, conversational)

1. Read the new file in `raw/`. If it has inline image references (from Obsidian Web Clipper), read the text first, then view the images that matter (see Image handling below).
2. Discuss the key takeaways with the user before writing anything — confirm what's actually worth capturing.
3. Write a `wiki/sources/<slug>.md` summary page: what the source claims, key findings/numbers, how it relates to temporal action quantization specifically.
4. Update or create the relevant `wiki/concepts/` and `wiki/entities/` pages — this is usually the bulk of the work. A single source may touch 5-10 pages. Flag explicitly when a new source contradicts or supersedes an existing claim (don't silently overwrite).
5. Update `wiki/synthesis/` pages if the source changes the overall picture or opens a new open question.
6. Update `wiki/index.md` with the new/changed pages.
7. Append an entry to `wiki/log.md`.

### Query

1. Read `wiki/index.md` first to find candidate pages.
2. Drill into the relevant `concepts/`, `entities/`, and `sources/` pages.
3. Synthesize an answer with citations back to source pages.
4. If the answer is substantial (a comparison, a synthesis, a new connection), offer to file it back into `wiki/synthesis/` as a new page rather than letting it disappear into chat history.
5. Append a short entry to `wiki/log.md` for non-trivial queries.

### Lint (run when asked, or proactively suggest after ~5-10 ingests)

Check for: contradictions between pages, claims superseded by newer sources, orphan pages, concepts mentioned in passing but lacking their own page, missing cross-references, and gaps that a targeted web search could fill. Propose findings before making changes.

## Image handling

Sources may include inline images (architecture diagrams, tokenization schemes, results tables as figures). Obsidian Web Clipper downloads these to `raw/assets/`. You can't read markdown-with-inline-images in one pass — read the source text first, then separately view images that are load-bearing for understanding (architecture figures, key result charts). Don't bother viewing purely decorative images.

## Log format

Each entry starts with a consistent prefix so it's greppable (`grep "^## \[" wiki/log.md | tail -5`):

```
## [YYYY-MM-DD] ingest | <Source Title>
- pages touched: ...
- key takeaway: ...

## [YYYY-MM-DD] query | <short question>
- filed to: wiki/synthesis/... (if applicable)

## [YYYY-MM-DD] lint
- findings: ...
```
