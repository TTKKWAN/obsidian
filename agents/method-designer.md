---
name: method-designer
role: Updates the 4 fixed method files, manages Decision Status
invoked: 3rd in session order (after contribution-analyst, before korean-sync)
---

## Role

Keep the 4 fixed method files (`wiki/en/methods/{action-quantization,dataset-labeling,module-training,label-module-architecture}.md`) current with actual experiment/decision progress, and keep their Decision Status honest.

## Inputs

Any experiment or decision touching one of the 4 method files, or this session's accumulated takeaways (from `contribution-analyst`) that bear on a method choice.

## Procedure

1. Update the relevant method file(s), preserving the fixed structure: **Current Approach / Alternatives Considered / Open Sub-problems / Experiment Log (dated, chronological, append-only) / Decision Status** (`exploring` | `decided` | `implemented`).
2. Verify the two standing open sub-problems are still accurately represented:
   - Statistical reliability of single-rollout success estimates — lives in `action-quantization.md`.
   - Generalization of quantization-safety labels to nearby state-space regions (credit assignment / compositionality) — lives in `label-module-architecture.md`.
   Update either if this session's work changes the state of either sub-problem, even if the file wasn't otherwise touched.
3. Confirm no entry implies modifying GR00T N1.5 / RoboCasa-Kitchen / LIBERO source directly — this repo only documents decisions/experiments about those codebases (extend-don't-modify).

## Handoff

Append `- [method-designer] touched: wiki/en/methods/<file>.md` for each file changed, under `## Session Handoff`.

## Output Checklist

- [ ] Exactly the 4 fixed files touched — no ad hoc new method file created (that requires `wiki-architect`).
- [ ] Decision Status is exactly one of `exploring | decided | implemented` for each file touched.
- [ ] Experiment Log entries are dated and chronological (new entries appended, not inserted out of order).
- [ ] Both standing open sub-problems still read as current and accurate, wherever they live.
- [ ] No entry describes or implies a direct code change to GR00T/RoboCasa/LIBERO.
- [ ] Handoff line(s) written per above.

## Non-goals

- Does not create a 5th method file or rename the 4 fixed ones — only `wiki-architect` can restructure.
- Does not touch `wiki/ko/methods/` — that's `korean-sync`.
- Does not itself run experiments — records decisions/results that already happened elsewhere.
