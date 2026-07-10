---
type: glossary
title: Notation Glossary
date: 2026-07-10
tags: [glossary]
twin: ../ko/glossary.md
last_synced: 2026-07-10
---

# Notation Glossary

Shared source of truth for notation used across `wiki/en/` and `wiki/ko/`. Symbols listed here are kept **untranslated** in the Korean twin — `korean-sync` must consult this file before translating any paper/method note so notation stays consistent across languages and across papers. New symbols introduced by a future ingested paper should be added here first.

Seeded from `raw/proposal.md` §11.

| Symbol / Term | Meaning | Where used |
|---|---|---|
| $\theta_i$ | Forward estimand $P(\text{success} \mid \text{quantize chunk } i, \text{obs}_i)$ — the training label | Data/label side |
| posterior | $P(\text{bit}=1 \mid \text{obs}, \text{success})$ — mixed with collection probability $p$, not a valid label; diagnostic only | Signal-existence check |
| $p$ | Per-chunk Bernoulli compression probability during mixed-quant rollout collection | Rollout collection |
| $D^*$ | Deployment $m$-distribution; fixed at the knee $p^*$ so the label is pinned to the deployment operating point | Label marginalization |
| $m / L$ | Number of co-quantized chunks in an episode / total chunks. $m \sim \text{Binomial}(L, p)$ | Stratification |
| $\kappa$ (kappa) | Inference-time routing threshold (operating point). Not a label — a deployment/eval threshold | Router deployment, sweep |
| $q$ | Compression ratio $H/T$. v1 coarse uses $q=4$ (`m4`), fallback $q=2$ (`m8`) | Decoder selection |
| contrast | $\sigma(\theta_{\text{fine}}) - \sigma(\theta_{\text{coarse}})$ — the module's per-chunk coarse-cost score | Routing decision |

> **Do not confuse $p$/$D^*$ with $\kappa$:** $p$/$D^*$ govern data collection and the label's operating point; $\kappa$ is the inference-time routing threshold. Different axes.

## Domain/project terms (not translated, kept as proper nouns or established acronyms in ko/ too)

GR00T N1.5, EagleBackbone, FlowmatchingActionHead, RoboCasa(-Kitchen), LIBERO, DexJoCo, Kinetix, ATQ (Adaptive Temporal Quantization), FAST, DCT+BPE.
