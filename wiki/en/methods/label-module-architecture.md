---
type: method
title: "Method: Label / Module Architecture"
date: 2026-07-10
tags: [action-quantization, credit-assignment, groot]
twin: ../../ko/methods/label-module-architecture.md
last_synced: 2026-07-10
---

# Method: Label / Module Architecture

See [glossary](../glossary.md) for notation and GR00T/decoder naming.

## Current Approach

**Backbone:** GR00T N1.5, frozen EagleBackbone + FlowmatchingActionHead. **Benchmarks:** RoboCasa-Kitchen (primary), DexJoCo, Kinetix (LIBERO also referenced in scope). **Collection checkpoint:** `groot_n1_5_bs64_moe_pyramid_K3_raw16_m8_m4_b_only_no_metaq_no_balance/checkpoint-60000` (`GR00T_N1_5_FairMoe`).

Trained decoders available: fine `action_decoder` ($T=16$), `m8_action_decoder` ($T=8$), `m4_full_action_decoder` ($T=4$). The checkpoint's built-in ATQ router is bypassed; the head is forced directly during mixed-quant rollout collection (see [dataset-labeling.md](dataset-labeling.md)).

**Naming trap (kept here verbatim since it's a real pitfall):** the user-facing "m2" (compression factor 2, $H16 \to T8$) corresponds to code `m8_action_decoder` (named for its *output* $T=8$). Code `m2_action_decoder` ($T=2$) is a different, harsher, unused decoder. **v1 coarse = $q4$ = `m4_full_action_decoder`.** Currently binary (main vs. one coarse decoder); 3-way routing (main/m8/m4) is a deliberate later extension, not yet implemented.

**Environment constraint:** the policy server (`gr00t` conda env) and the sim client (`robocasa_gr00t` conda env) communicate over zmq and cannot share a process. All masked-rollout collection runs as the client.

**Extend-don't-modify boundary:** GR00T N1.5, RoboCasa-Kitchen, and LIBERO codebases are referenced and documented here only. No workflow in this wiki (or its agents) modifies those codebases directly — architecture decisions about them are recorded as documentation, and any actual code changes happen in those projects' own repos, not here.

## Alternatives Considered

- **3-way routing (main/m8/m4) now** — deferred; binary routing (main vs. one coarse) is the current scope, 3-way is an intentional later extension once binary routing is validated.
- **Modifying the GR00T head/router directly** — rejected; violates extend-don't-modify. Client-side-only deployment (see [module-training.md](module-training.md)) is the chosen path specifically to avoid this.

## Open Sub-problems

**Standing sub-problem 2 (always keep current): generalization of quantization-safety labels to nearby state-space regions — credit assignment / compositionality (hazard ④).** When multiple chunks in an episode are co-quantized together, episode success is a property of the *joint* set of compressed chunks, not attributable cleanly to any one chunk. $m$-stratification (see [dataset-labeling.md](dataset-labeling.md), hazard ②) resolves "how many chunks were co-quantized" but leaves "which specific partner combination mattered" as residual variance. The clean fix is a **paired counterfactual**: branch chunk $i$ alone (fine vs. coarse, same downstream) from an on-policy mixed-quant snapshot, measuring $\theta_i$ per-chunk with everything else held fixed. This is 2x the rollout cost, so it's a **fallback**, triggered only if `m4`-aggregate data (once collected) also turns out flat or unreliable near contact-rich chunks — not the default approach.

Important distinction from a discarded approach: the paired-counterfactual's snapshot/restore *engineering* is reusable from the old (discarded) clean-fine counterfactual method — only the observation distribution differs (on-policy mixed-quant here, vs. off-distribution clean-fine there, which is why that method was discarded; see [action-quantization.md](action-quantization.md) Alternatives Considered).

## Experiment Log

_(none yet — this file tracks architecture/credit-assignment decisions specifically; see [dataset-labeling.md](dataset-labeling.md) and [module-training.md](module-training.md) for the `m8`/`m4` experiment entries already logged)_

## Decision Status

`exploring` — paired counterfactual remains a fallback, not yet needed or implemented; will move to `decided` (adopt) or stay `exploring` (still sufficient without it) once `m4` aggregate data is examined for flatness near contact-boundary chunks.
