---
type: method
title: "Method: Action Quantization"
date: 2026-07-10
tags: [action-quantization, temporal-compression, chunking]
twin: ../../ko/methods/action-quantization.md
last_synced: 2026-07-10
---

# Method: Action Quantization

See [glossary](../glossary.md) for notation ($q$, chunk, coarse/fine decoder).

## Current Approach

Temporal-horizon compression only — quantizing *when* the policy re-plans, not action-token discretization (FAST/DCT+BPE), not weight/activation precision quantization, and not vision-input compression (per `raw/proposal.md` §1 scope defense). "Coarse execution" of a chunk means calling an already-trained coarse decoder (v1: `m4_full_action_decoder`, $q=4$) — never a post-hoc block-aggregation of the fine decoder's output (that's a discarded approach, see Alternatives Considered).

Compression ratio $q = H/T$ where $H=16$ is the fine chunk horizon. v1 coarse decoder targets $q=4$ (code name `m4_full_action_decoder`, output $T=4$).

## Alternatives Considered

- **$q=2$ (`m8_action_decoder`, code-name mismatch: "m8" = factor-2 compression, output $T=8$)** — tested first; too weak. Per-chunk degradation from $q=2$ compression falls below the episode-failure threshold on RoboCasa, so risky chunks getting compressed doesn't reliably fail the episode → the negative-labeling mechanism (see [dataset-labeling.md](dataset-labeling.md)) never fires → flat/uninformative labels. Confirmed as the weakest measured cell (−5.6pp vs. ATQ's own −18 to −29pp range at other quantization levels). Kept only as a fallback if $q=4$ proves too aggressive (collapses success rate too much).
- **Post-hoc block-aggregation "coarse"** (discarded, do not reintroduce) — synthesizing a coarse output by block-summing the fine decoder's 16-step output. This was a train/serve mismatch: it was the *training target* for the coarse decoder, never how coarse execution should happen at rollout time. Caused router train/serve mismatch when it was used as the deployed coarse-execution mechanism (forced by an early checkpoint lacking a real coarse decoder).
- **Old (clean-fine) counterfactual labeling** (discarded, do not reintroduce) — branching a chunk from a *clean fine* episode rather than an on-policy mixed-quant one. Produces off-distribution observations relative to deployment. The snapshot/restore engineering is reusable for the paired-counterfactual fallback (see [label-module-architecture.md](label-module-architecture.md)), but the clean-fine variant itself is not.

## Open Sub-problems

**Standing sub-problem 1 (always keep current): statistical reliability of single-rollout success estimates.** Each per-chunk label comes from exactly one noisy 0/1 rollout outcome (proposal.md §5, hazard ③) — the same observation is never revisited. The module is expected to generalize via function smoothness across *nearby* observations, which is known to hold except near contact-boundary chunks (hazard ④, credit-assignment territory — tracked in [label-module-architecture.md](label-module-architecture.md)). No quantitative bound on this reliability exists yet; this is the most direct link between "how much rollout data is enough" and "how confident can the router be."

Secondary open items:
- Whether $q=4$ turns out to still be too weak or too strong once feature collection (job `406006`) completes — the fallback to $q=2$ remains live until that's resolved.

## Experiment Log

- **2026-07-09** — `m8`/$q=2$ pipeline validated end-to-end; confirmed as negative control (flat `contrast_gap ≈ 0`), consistent with $q=2$ being the weakest measured compression cell. Deployment client, kappa calibration, eval sweep, and summarize scripts all staged/submitted for this run.
- **2026-07-09 (ongoing)** — `m4`/$q=4$ feature collection in progress (job `406006`) — this is the run expected to show a real (non-flat) signal, per ATQ's own tables showing non-quantizable chunks exist at this compression level.

## Decision Status

`exploring` — $q=4$ is the current working hypothesis (v1 coarse decoder choice) but not yet validated; status moves to `decided` once `m4` feature collection + module training shows a non-flat `contrast_gap`.
