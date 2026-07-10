---
type: method
title: "Method: Action Quantization"
date: 2026-07-10
tags: [action-quantization, temporal-compression, chunking]
twin: ../../en/methods/action-quantization.md
last_synced: 2026-07-10
---

# Method: Action Quantization

표기법($q$, chunk, coarse/fine decoder)은 [용어집](../glossary.md) 참고.

## Current Approach

오직 시간축(temporal-horizon) 압축만 다룬다 — *언제* policy가 재계획하는지를 quantize하는 것이지, action-token discretization(FAST/DCT+BPE)도, weight/activation precision quantization도, vision-input 압축도 아니다(`raw/proposal.md` §1의 scope 방어 참고). Chunk의 "coarse 실행"이란 이미 학습된 coarse decoder를 호출하는 것을 의미한다(v1: `m4_full_action_decoder`, $q=4$) — fine decoder 출력의 post-hoc block-aggregation이 절대 아니다(폐기된 방식, 아래 Alternatives Considered 참고).

압축비 $q = H/T$, 여기서 $H=16$은 fine chunk horizon. v1 coarse decoder는 $q=4$를 목표로 한다(code name `m4_full_action_decoder`, 출력 $T=4$).

## Alternatives Considered

- **$q=2$ (`m8_action_decoder`, 이름 함정: "m8"은 압축 factor 2, 출력 $T=8$을 의미)** — 먼저 테스트했으나 너무 약함. $q=2$ 압축으로 인한 per-chunk 열화가 RoboCasa에서 episode 실패 임계값보다 작아, 위험한 chunk를 압축해도 episode가 신뢰성 있게 실패하지 않음 → negative-labeling 메커니즘([dataset-labeling.md](dataset-labeling.md) 참고)이 발화하지 않음 → flat/무정보 label. 측정된 셀 중 가장 약한 것으로 확인됨(다른 quantization 레벨의 −18~−29pp 범위 대비 −5.6pp). $q=4$가 너무 공격적인 것으로 판명될 경우(성공률을 과도하게 붕괴시킬 경우)의 fallback으로만 유지.
- **Post-hoc block-aggregation "coarse"** (폐기, 재도입 금지) — fine decoder의 16-step 출력을 block-sum해 coarse 출력을 합성하는 방식. Train/serve mismatch였다: 이는 coarse decoder의 *학습 target*이었을 뿐, rollout 시점에 coarse를 실행하는 방법이 절대 아니었다. 실제 coarse-실행 메커니즘으로 배포됐을 때 라우터의 train/serve mismatch를 유발함(실제 coarse decoder가 없던 초기 checkpoint 때문에 강제됐던 방식).
- **Old (clean-fine) counterfactual labeling** (폐기, 재도입 금지) — on-policy mixed-quant가 아니라 *clean fine* episode에서 chunk를 분기하는 방식. 배포 분포 대비 off-distribution인 obs를 만들어냄. Snapshot/restore 엔지니어링은 paired-counterfactual fallback([label-module-architecture.md](label-module-architecture.md) 참고)에서 재사용 가능하지만, clean-fine 방식 자체는 재사용하지 않는다.

## Open Sub-problems

**표준 sub-problem 1 (항상 최신 상태로 유지): single-rollout success estimate의 통계적 신뢰도.** 각 per-chunk label은 노이즈가 섞인 0/1 rollout 결과 하나에서만 나온다(proposal.md §5, hazard ③) — 같은 observation은 다시 등장하지 않는다. 모듈은 함수의 smoothness를 통해 *인접* observation들에 대해 일반화할 것으로 기대되며, 이는 contact-boundary chunk(④, credit-assignment 영역 — [label-module-architecture.md](label-module-architecture.md)에서 추적) 근처를 제외하면 성립하는 것으로 알려져 있다. 이 신뢰도에 대한 정량적 한계는 아직 존재하지 않는다; 이는 "얼마나 많은 rollout 데이터가 충분한가"와 "라우터가 얼마나 확신할 수 있는가"를 직접 연결하는 지점이다.

부차적 open item:
- Feature collection(job `406006`) 완료 후 $q=4$가 여전히 너무 약하거나 너무 강한 것으로 판명될지 여부 — 이것이 해결되기 전까지 $q=2$ fallback은 유효하게 남아있다.

## Experiment Log

- **2026-07-09** — `m8`/$q=2$ 파이프라인 end-to-end 검증 완료; negative control로 확인됨(flat `contrast_gap ≈ 0`), $q=2$가 측정된 압축 셀 중 가장 약하다는 것과 일치. 이 run을 위한 배포 client, kappa calibration, eval sweep, summarize 스크립트 모두 staged/submitted.
- **2026-07-09 (진행 중)** — `m4`/$q=4$ feature collection 진행 중(job `406006`) — ATQ 논문 자체의 테이블이 이 압축 레벨에서 non-quantizable chunk가 존재함을 보여주므로, 진짜(non-flat) signal을 보여줄 것으로 예상되는 run.

## Decision Status

`exploring` — $q=4$가 현재 작업 가설(v1 coarse decoder 선택)이지만 아직 검증되지 않음; `m4` feature collection + 모듈 학습이 non-flat `contrast_gap`을 보여주면 `decided`로 이동.
