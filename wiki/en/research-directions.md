---
type: research-directions
title: Research Directions
date: 2026-07-10
tags: [action-quantization, success-grounded-labeling]
twin: ../ko/research-directions.md
last_synced: 2026-07-10
---

# Research Directions

Append-only, date-stamped. New entries are added at the bottom of the dated log; past entries are never edited, only superseded by a newer entry that says so explicitly. See [glossary](glossary.md) for notation.

## Open Questions (running list)

These are updated in place (not append-only) as understanding sharpens — they are the two standing open sub-problems this project must always keep current, per `raw/proposal.md` §5 and §9:

1. **Statistical reliability of single-rollout success estimates.** Each `(obs, bit, m)` data point comes from one noisy 0/1 rollout outcome — the same obs never recurs. Current mitigation (per proposal.md §5, hazard ③): rely on function smoothness so the module averages over *nearby* observations. Open: how to quantify/bound the reliability of this smoothing, especially near contact boundaries (hazard ④) where it's known to break down.
2. **Generalization of quantization-safety labels to nearby state-space regions.** Tied to hazard ④ (credit assignment / compositionality): when multiple chunks are co-quantized, success is a property of the *joint*, and $m$-stratification only resolves "how many" were co-quantized, not "which specific partner." The clean fix (paired counterfactual, proposal.md §9) is a costly fallback, not the default. Open: whether $m$-stratify + smoothness is sufficient in practice, or whether paired counterfactual becomes necessary once `m4` aggregate data is examined.

## Dated Log

### 2026-07-10 — Vault bootstrap; seeding from proposal.md

Initial entry, seeded directly from `raw/proposal.md` (the project's canonical spec) as part of the wiki redesign bootstrap (see `.omc/plans/vla-wiki-vault-redesign.md`).

- **Project:** Success-Grounded Quantization Signal Router. A per-chunk module that decides, right before executing an action chunk, whether it is safe to dispatch to a coarse (temporally-compressed, $q=4$, `m4_full_action_decoder`) vs. fine (`action_decoder`, $T=16$) decoder — trained on labels grounded in actual mixed-quant rollout success, not ATQ's reconstruction-fit proxy.
- **Why relevant:** ATQ (Adaptive Temporal Quantization) is the reference/inspiration paper; its own Limitations section proposes exactly this (rollout-success-grounded quantizability supervision) as future work. This project realizes that future work, reusing ATQ-style decoders unchanged and swapping only the router's training signal.
- **Current status (per proposal.md §8, as of 2026-07-09):** `m8`/`q2` pipeline validated end-to-end as a negative control (flat contrast_gap, as predicted — RoboCasa×q2 is the weakest cell). The real signal test is `m4`/`q4` (feature collection in progress, job `406006`).
- **Backlog:** 7 raw papers not yet ingested — see [index.md](index.md) for status. `Action_Quantization (12).pdf` is presumed to be the ATQ reference paper itself (to be confirmed on ingestion); `Action_Quantization_supplementary_restore (2).pdf` its supplementary material.

## Sync Log

_(append-only; `korean-sync` writes exactly one line here every session it runs, including no-op runs — see `agents/korean-sync.md`)_

- [2026-07-10] korean-sync: synced 8 file(s) (bootstrap: index.md, tags.md, research-directions.md, glossary.md, methods/action-quantization.md, methods/dataset-labeling.md, methods/module-training.md, methods/label-module-architecture.md)
