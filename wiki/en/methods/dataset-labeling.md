---
type: method
title: "Method: Dataset Labeling"
date: 2026-07-10
tags: [success-grounded-labeling, mixed-quant-rollout]
twin: ../../ko/methods/dataset-labeling.md
last_synced: 2026-07-10
---

# Method: Dataset Labeling

See [glossary](../glossary.md) for notation ($\theta_i$, posterior, $p$, $D^*$, $m/L$).

## Current Approach

Labels are grounded in **actual rollout success**, not a reconstruction-fit proxy (the key departure from ATQ). Pipeline (per `raw/proposal.md` §4):

1. **Mixed-quant rollout.** During GR00T closed-loop execution, each chunk's head is forced by an i.i.d. Bernoulli($p$) coin: bit=0 → fine (main, $T=16$), bit=1 → coarse (learned coarse decoder, v1 `m4`). The checkpoint's built-in ATQ router is bypassed. Boundary observations therefore come from the **on-policy mixed-quant** distribution, not a clean-fine distribution.
2. **Success filter.** Only successful episodes are kept; each surviving episode contributes per-chunk `(obs, bit, m)` tuples. **Failed rollouts are discarded — never used as training data.** Every observation is grounded only via success.
3. **Negative mechanism.** A chunk that is genuinely risky to compress, if compressed, tends to fail its episode — so it gets filtered out by the success filter. What survives in the success set is disproportionately the safely-compressible chunks; *absence* in the success set is what creates the negative signal. ⚠️ This mechanism only fires when compression is strong enough to actually cause episode failure (see [action-quantization.md](action-quantization.md) — this is exactly why $q=2$ was too weak).
4. **Label = $\theta_i$.** Each chunk's outcomes are stratified by $m$ (co-quantization count) and reweighted to the deployment distribution $D^*$, fixed at the knee $p^*$ (the success–speed tradeoff's best operating point) — see below.
5. Train a small obs→$\theta$ module on these labels; deploy as router; score on success–speed tradeoff vs. baselines (see [module-training.md](module-training.md)).

### Estimand (must be read precisely)

The module predicts the **forward** interventional probability $\theta_i = P(\text{success} \mid \text{quantize chunk } i, \text{obs}_i)$ — **not** the posterior $P(\text{quantized} \mid \text{obs}_i, \text{success})$. The posterior is entangled with the collection rate $p$ (`P(bit=1|obs,succ) = θ_i·p / P(succ|obs)`), so it shifts depending on how the data happened to be collected. Forward $\theta_i$ strips $p$ out and is the valid label; the posterior is used only as a diagnostic for "does a signal exist at all," never as a training target.

### Co-quantization dependency and marginalization

A chunk's success depends on how many *other* chunks in the same episode were also compressed ($m$). Since compression is i.i.d. Bernoulli($p$) per chunk, $m \sim \text{Binomial}(L, p)$ ($L$ = chunks per episode). The full label is a curve over $m$; it's marginalized against a chosen deployment distribution $D^*$ to collapse to one scalar label per chunk: $\theta_i = \sum_m P(m \mid D^*) \cdot \theta_i(m)$. **Choosing $D^*$ = choosing the operating compression rate.** $D^*$ is fixed at the knee $p^*$ (best success–speed tradeoff point) so the label is pinned to the actual deployment operating point rather than whatever $p$ the data happened to be collected at.

## Alternatives Considered

- **Naive averaging over all collected $m$** — rejected; the estimate would be dragged around by whatever collection $p$ happened to be used, rather than reflecting a fixed deployment operating point. See hazard ② below.
- **Using the posterior directly as a label** — rejected; conflates $\theta_i$ with the collection rate $p$ (see Estimand above). Retained only as a diagnostic for signal existence.

## Open Sub-problems

Hazard table (per `raw/proposal.md` §5), current status of each:

| # | Hazard | Handling | Status |
|---|---|---|---|
| ① | A chunk staying fine in the data = "never tried," not "can't be compressed" | Success filter aggregates across many masks; risky-chunk-compressed rollouts fail → drop → that chunk mostly survives as fine → low $\theta_i$ | Only fires when compression is strong enough (this is why $q=2$/`m8` was too weak; see [action-quantization.md](action-quantization.md)) |
| ② | Success depends on co-quantization count $m$ → naive averaging is dragged around by collection $p$ | Stratify by $m$, reweight to $D^*$ = knee $p$ | Handled (data-hungry; $D^*$ fixed at knee) |
| ③ | One noisy 0/1 label per observation, same observation never recurs | Module averages over *nearby* observations via function smoothness; needs broad obs coverage | Works except near contact boundaries (④) — tracked as standing sub-problem 1 in [action-quantization.md](action-quantization.md) |
| ④ | Credit assignment / compositionality — success is a joint property when multiple chunks are co-quantized | ② handles "how many"; "which specific partner" remains residual. Clean fix = paired counterfactual (costly, fallback only) | **OPEN** — tracked as standing sub-problem 2 in [label-module-architecture.md](label-module-architecture.md) |

## Experiment Log

- **2026-07-09** — Confirmed that timeout-failures are genuine, unrecoverable failures (not a horizon-confound artifact): coarse execution's step savings cannot rescue an episode that would have succeeded but timed out under fine execution — no such episodes were found. This validates binary success as a sound label without needing a horizon adjustment. Remaining hygiene item: use $m/L$ or per-chunk rates rather than raw $m$ counts in analysis.

## Decision Status

`decided` for the estimand/label definition ($\theta_i$, forward not posterior) and the $m$-stratify + $D^*$-at-knee marginalization approach. `exploring` for hazard ④ (credit assignment) — still open.
