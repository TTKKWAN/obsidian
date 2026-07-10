---
type: method
title: "Method: Module Training"
date: 2026-07-10
tags: [action-quantization, router]
twin: ../../en/methods/module-training.md
last_synced: 2026-07-10
---

# Method: Module Training

표기법($\kappa$, contrast)은 [용어집](../glossary.md) 참고.

## Current Approach

작은 모듈이 obs → $\theta$(quantization 하에서의 per-chunk 성공확률 추정치)를 매핑하며, [dataset-labeling.md](dataset-labeling.md)에서 설명한 label로 학습된다. 이는 체크포인트 내장 ATQ router를 대체하는 **client-side-only 라우터**로 배포된다:

- Server는 fine·coarse 두 branch 출력과 ATQ router 자체의 feature를 모두 반환한다 — head나 server 수정 없음(GR00T N1.5에 대한 extend-don't-modify 유지).
- Client가 학습된 모듈을 통해 어느 branch를 쓸지 고른다.
- **Routing 규칙:** `contrast = σ(θ_fine) − σ(θ_coarse)`; `bit = int(contrast ≤ κ)`.
- **κ(kappa)는 operating point:** κ가 낮을수록 → coarse dispatch 증가(빠름, 위험↑); κ가 높을수록 → 대부분 fine(안전, 느림). Success–speed tradeoff 곡선을 그리기 위해 sweep한다. 서로 다른 모듈 간 곡선을 비교 가능하게 하기 위해 coarse-rate 분위수 {0.1, 0.3, 0.5, 0.7, 0.9}로 κ를 calibrate한다.

### 평가 프로토콜

- **Baseline:** all-fine(성공률 상한), all-coarse(하한), random-$p$(공짜 기준선), 그리고 — 이겨야 할 대상인 — **ATQ 자신의 reconstruction-fit 라우터**, 모두 동일한 dispatch 규칙으로 비교.
- **승리 조건:** 동일 speedup에서 더 높은 성공률, 또는 동일 성공률에서 더 큰 speedup.

## Alternatives Considered

- **Server-side / head 수정을 통한 라우팅** — 기각; GR00T N1.5 체크포인트와 head 아키텍처에 대한 extend-don't-modify 제약을 위반한다. Client-side-only 라우팅은 배포된 policy server를 수정하지 않고 유지한다.
- **Sweep 없는 단일 고정 κ를 주 평가 방법으로 사용** — 기각; 단일 지점으로는 실제 비교 대상인 ATQ 라우터 대비 진짜 success–speed tradeoff 승리를 입증할 수 없다.

## Open Sub-problems

- AUC 수준의 θ_fine/θ_coarse 품질(`m8` negative-control run에서 측정된 ~0.85–0.90, task/phase 난이도 baseline을 반영)이 상한인지, 아니면 `m4` 학습 데이터가 이를 바꾸는지 여부.
- 서로 다른 $p$/$D^*$ operating point로 학습된 모듈들 사이에서 κ-분위수 binning의 calibration 안정성 — 아직 stress-test 안 됨.

## Experiment Log

- **2026-07-09** — `m8`/$q=2$ 데이터로 $p \in \{0.2, 0.4, 0.6, 0.8\}$에서 모듈 4개 학습. θ_fine/θ_coarse AUC ~0.85–0.90(압축 신호가 아니라 task/phase 난이도와 일치)이지만 `contrast_gap ≈ 0` — `m8`이 사용 가능한 라우팅 signal을 만들기엔 너무 약한 압축 레벨임을 확인(예상대로의 negative control).
- **2026-07-09** — 배포 client(`robocasa_service_qs_router.py`), κ calibration, eval sweep, summarize 스크립트 모두 staged/submitted; 파이프라인으로서는 검증됨(단, `m8` 모듈이 설계상 flat이므로 signal로서는 아직 검증 안 됨).

## Decision Status

Routing 규칙(contrast + κ threshold)과 평가 프로토콜(baseline, 승리 조건)은 `decided`. `m4`로 학습된 모듈이 진짜(non-flat) signal을 보여줄지는 `exploring` — 파이프라인의 다음 진짜 테스트.
