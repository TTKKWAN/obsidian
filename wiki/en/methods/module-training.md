---
type: method
title: "Method: Module Training"
date: 2026-07-10
tags: [action-quantization, router]
twin: ../../ko/methods/module-training.md
last_synced: 2026-07-10
---

# Method: Module Training

See [glossary](../glossary.md) for notation ($\kappa$, contrast).

## Current Approach

A small module maps obs → $\theta$ (per-chunk success-probability estimate under quantization), trained on the labels described in [dataset-labeling.md](dataset-labeling.md). It is deployed as a **client-side-only router**, replacing the checkpoint's built-in ATQ router:

- The server returns both fine and coarse branch outputs plus the ATQ router's own feature — no head or server modification (extend-don't-modify preserved for GR00T N1.5).
- The client picks which branch to use via the trained module.
- **Routing rule:** `contrast = σ(θ_fine) − σ(θ_coarse)`; `bit = int(contrast ≤ κ)`.
- **κ (kappa) is the operating point:** lower κ → more coarse dispatch (faster, riskier); higher κ → mostly fine (safer, slower). Swept to trace the success–speed tradeoff curve. Coarse-rate quantiles {0.1, 0.3, 0.5, 0.7, 0.9} are used to calibrate κ so curves are comparable across different modules.

### Evaluation protocol

- **Baselines:** all-fine (upper bound on success rate), all-coarse (lower bound), random-$p$ (free baseline), and — the one to beat — **ATQ's own reconstruction-fit router**, all compared under the identical dispatch rule.
- **Win condition:** higher success rate at equal speedup, or greater speedup at equal success rate.

## Alternatives Considered

- **Server-side / head modification for routing** — rejected; violates the extend-don't-modify constraint on GR00T N1.5's checkpoint and head architecture. Client-side-only routing keeps the deployed policy server unmodified.
- **Single fixed κ rather than a swept operating curve** — rejected as the primary evaluation method; a single point can't demonstrate a genuine success–speed tradeoff win over ATQ's router, which is the actual comparison target.

## Open Sub-problems

- Whether AUC-level θ_fine/θ_coarse quality (~0.85–0.90 measured on the `m8` negative-control run, reflecting task/phase difficulty baseline) is a ceiling or whether `m4` training data changes this.
- Calibration stability of κ-quantile binning across modules trained on different $p$/$D^*$ operating points — not yet stress-tested.

## Experiment Log

- **2026-07-09** — 4 modules trained on `m8`/$q=2$ data at $p \in \{0.2, 0.4, 0.6, 0.8\}$. θ_fine/θ_coarse AUC ~0.85–0.90 (consistent with task/phase difficulty, not a compression signal) but `contrast_gap ≈ 0` — confirms `m8` is too weak a compression level to produce a usable routing signal (negative control, as predicted).
- **2026-07-09** — Deployment client (`robocasa_service_qs_router.py`), κ calibration, eval sweep, and summarize scripts all staged/submitted; validated as a pipeline (not yet as a signal, since the `m8` modules are flat by design).

## Decision Status

`decided` for the routing rule (contrast + κ threshold) and evaluation protocol (baselines, win condition). `exploring` for whether `m4`-trained modules will show a real (non-flat) signal — this is the pipeline's next real test.
