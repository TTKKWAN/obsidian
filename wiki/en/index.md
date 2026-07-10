---
type: index
title: Index
date: 2026-07-10
tags: [meta]
twin: ../ko/index.md
last_synced: 2026-07-10
---

# Index

Catalog of the wiki. Maintained by `wiki-architect` (see `agents/wiki-architect.md`); updated at the end of every session that touches `wiki/`.

## Project Focus

This vault documents research on **success-grounded quantization signal routing** for VLA action chunks — see [raw/proposal.md](../../raw/proposal.md) for the canonical spec, and [research-directions.md](research-directions.md) for the evolving research direction and standing open questions.

## Papers

| Slug | Title | Status | Tags | Link |
|---|---|---|---|---|
| `atq-adaptive-temporal-quantization` | *(provisional — presumed ATQ reference paper, "Adaptive Temporal Quantization"; confirm on ingest)* | backlog | `atq`, `action-quantization` | [raw/papers/atq-adaptive-temporal-quantization/original.pdf](../../raw/papers/atq-adaptive-temporal-quantization/original.pdf) |
| `atq-supplementary` | *(provisional — supplementary material for the ATQ paper above; confirm on ingest)* | backlog | `atq` | [raw/papers/atq-supplementary/original.pdf](../../raw/papers/atq-supplementary/original.pdf) |
| `dynamic-test-time-compute-control-policy` | *(provisional, from filename "Dynamic Test-Time Compute Scaling in Control Policy")* | backlog | `test-time-compute` | [raw/papers/dynamic-test-time-compute-control-policy/original.pdf](../../raw/papers/dynamic-test-time-compute-control-policy/original.pdf) |
| `moh` | *(provisional, from filename "MoH")* | backlog | _(tbd on ingest)_ | [raw/papers/moh/original.pdf](../../raw/papers/moh/original.pdf) |
| `aac` | *(provisional, from filename "AAC")* | backlog | _(tbd on ingest)_ | [raw/papers/aac/original.pdf](../../raw/papers/aac/original.pdf) |
| `dynamic-execution-horizon-prediction` | *(provisional, from filename "Dynamic Execution Horizon Prediction")* | backlog | `execution-horizon` | [raw/papers/dynamic-execution-horizon-prediction/original.pdf](../../raw/papers/dynamic-execution-horizon-prediction/original.pdf) |
| `vlash` | *(provisional, from filename "vlash")* | backlog | `vla` | [raw/papers/vlash/original.pdf](../../raw/papers/vlash/original.pdf) |

All 7 slugs are provisional until `data-collector` actually opens and ingests each paper (see `agents/data-collector.md`); titles/tags will be corrected at that point.

## Methods

| File | Decision Status |
|---|---|
| [action-quantization.md](methods/action-quantization.md) | exploring |
| [dataset-labeling.md](methods/dataset-labeling.md) | exploring |
| [module-training.md](methods/module-training.md) | exploring |
| [label-module-architecture.md](methods/label-module-architecture.md) | exploring |

## Research Directions

See [research-directions.md](research-directions.md). Standing open sub-problems (always kept current there):
1. Statistical reliability of single-rollout success estimates.
2. Generalization of quantization-safety labels to nearby state-space regions.

## Glossary

See [glossary.md](glossary.md) for notation ($\theta$, $\kappa$, $p$, $D^*$, $m/L$, $q$, ...).

## Tags

See [tags.md](tags.md) for the full taxonomy and governance rule.
