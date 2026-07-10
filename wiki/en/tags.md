---
type: tags
title: Tag Taxonomy
date: 2026-07-10
tags: [meta]
twin: ../ko/tags.md
last_synced: 2026-07-10
---

# Tag Taxonomy

Single source of truth for tags used in frontmatter across `wiki/en/` (and mirrored `wiki/ko/`). **A new tag must be added here before its first use in any file's frontmatter.** `data-collector` may provisionally add a tag under a `(pending review)` marker; only `wiki-architect` resolves (confirms or rejects) pending markers, at end of session.

## Domain / environment tags

- `simulation`
- `real-robot`
- `dataset`
- `web`
- `mobile`
- `macos`, `ubuntu`, `windows` (kept in the taxonomy per project convention even though not yet relevant to this robotics-only project)

## Research tags (action-quantization project)

- `vla` — vision-language-action models generally
- `action-quantization` — umbrella tag for this project's core topic
- `temporal-compression` — horizon/time-axis compression specifically (as opposed to weight/activation/vision quantization)
- `tokenization`
- `discretization`
- `flow-matching`
- `diffusion-policy`
- `chunking`
- `action-chunk`
- `router` — dispatch/routing modules (e.g. the quantization signal router)
- `success-grounded-labeling` — labels derived from actual rollout success rather than a reconstruction proxy
- `mixed-quant-rollout`
- `credit-assignment` — compositionality / joint-success attribution across co-quantized chunks
- `groot` — GR00T N1.5 backbone
- `robocasa`
- `libero`
- `dexjoco`
- `kinetix`
- `atq` — Adaptive Temporal Quantization, the reference paper this project extends
- `test-time-compute`
- `execution-horizon`

## Governance rule

A tag not listed here may not appear in any file's frontmatter without first being added here (or flagged `(pending review)` by `data-collector` and resolved by `wiki-architect`, per `agents/wiki-architect.md`).
