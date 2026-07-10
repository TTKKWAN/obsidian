---
type: method
title: "Method: Dataset Labeling"
date: 2026-07-10
tags: [success-grounded-labeling, mixed-quant-rollout]
twin: ../../en/methods/dataset-labeling.md
last_synced: 2026-07-10
---

# Method: Dataset Labeling

표기법($\theta_i$, posterior, $p$, $D^*$, $m/L$)은 [용어집](../glossary.md) 참고.

## Current Approach

Label은 reconstruction-fit proxy가 아니라 **실제 rollout success**로 grounding된다(ATQ와의 핵심 차이). 파이프라인(`raw/proposal.md` §4 기준):

1. **Mixed-quant rollout.** GR00T closed-loop 실행 중, chunk마다 i.i.d. Bernoulli($p$) 동전으로 head를 강제한다: bit=0 → fine(main, $T=16$), bit=1 → coarse(학습된 coarse decoder, v1 `m4`). 체크포인트 내장 ATQ router는 우회된다. 따라서 경계 observation은 clean-fine 분포가 아니라 **on-policy mixed-quant** 분포에서 나온다.
2. **Success filter.** 성공한 episode만 남긴다; 살아남은 각 episode가 per-chunk `(obs, bit, m)` 튜플을 제공한다. **실패한 rollout은 버린다 — 절대 학습 데이터로 쓰지 않는다.** 모든 observation은 오직 success를 통해서만 grounding된다.
3. **Negative mechanism.** 진짜로 압축에 위험한 chunk는, 압축되면 해당 episode를 실패시키는 경향이 있어 success filter에 의해 걸러진다. Success set에 남는 것은 안전하게 압축 가능한 chunk 쪽으로 편향된다 — success set에서의 *부재*가 negative signal을 만든다. ⚠️ 이 메커니즘은 압축이 실제로 episode 실패를 유발할 만큼 충분히 강할 때만 발화한다([action-quantization.md](action-quantization.md) 참고 — $q=2$가 왜 너무 약했는지 바로 이 이유 때문).
4. **Label = $\theta_i$.** 각 chunk의 결과를 $m$(co-quantization 개수)으로 stratify하고, 배포 분포 $D^*$(success–speed tradeoff가 가장 좋은 지점인 knee $p^*$에 고정)로 reweight한다 — 아래 참고.
5. 이 label들로 작은 obs→$\theta$ 모듈을 학습하고, 라우터로 배포하고, baseline 대비 success–speed tradeoff로 채점한다([module-training.md](module-training.md) 참고).

### Estimand (정확히 읽어야 함)

모듈이 예측하는 것은 개입(intervention) 하의 **forward** 확률 $\theta_i = P(\text{success} \mid \text{quantize chunk } i, \text{obs}_i)$이다 — posterior $P(\text{quantized} \mid \text{obs}_i, \text{success})$가 **아니다**. Posterior는 수집률 $p$와 얽혀 있어(`P(bit=1|obs,succ) = θ_i·p / P(succ|obs)`), 데이터가 어떤 $p$로 수집됐는지에 따라 값이 흔들린다. Forward $\theta_i$는 $p$를 벗겨낸, 유효한 label이다; posterior는 오직 "signal이 존재하는가"를 확인하는 진단용으로만 쓰이며 학습 target으로는 절대 쓰이지 않는다.

### Co-quantization 의존성과 marginalization

한 chunk의 성공은 같은 episode에서 *다른* 몇 개의 chunk가 함께 압축됐는지($m$)에 달려 있다. 압축이 chunk별 i.i.d. Bernoulli($p$)이므로 $m \sim \text{Binomial}(L, p)$($L$ = episode당 chunk 수). 전체 label은 $m$에 대한 곡선이며, 선택한 배포 분포 $D^*$로 marginalize해 chunk당 하나의 스칼라 label로 접는다: $\theta_i = \sum_m P(m \mid D^*) \cdot \theta_i(m)$. **$D^*$를 고르는 것 = operating 압축률을 고르는 것.** $D^*$는 knee $p^*$(success–speed tradeoff가 가장 좋은 지점)에 고정되어, label이 데이터가 우연히 수집된 $p$가 아니라 실제 배포 operating point에 묶이도록 한다.

## Alternatives Considered

- **수집된 모든 $m$에 대한 naive averaging** — 기각; 이 추정치는 고정된 배포 operating point를 반영하는 대신 데이터 수집 시 우연히 사용된 $p$에 끌려다니게 된다. 아래 hazard ② 참고.
- **Posterior를 직접 label로 사용** — 기각; $\theta_i$를 수집률 $p$와 혼동한다(위 Estimand 참고). Signal 존재 여부를 확인하는 진단용으로만 유지.

## Open Sub-problems

Hazard 테이블(`raw/proposal.md` §5 기준), 각 항목의 현재 상태:

| # | Hazard | 처리 | 상태 |
|---|---|---|---|
| ① | Chunk가 데이터에서 fine으로 남음 = "안 눌러봄"이지 "압축 불가"가 아님 | Success filter가 여러 mask를 aggregate; 위험-chunk-압축 rollout은 실패 → drop → 그 chunk는 대부분 fine으로 생존 → 낮은 $\theta_i$ | 압축이 충분히 강할 때만 발화(그래서 $q=2$/`m8`이 너무 약했음 — [action-quantization.md](action-quantization.md) 참고) |
| ② | 성공이 co-quantization 개수 $m$에 의존 → naive averaging은 수집 $p$에 끌려감 | $m$으로 stratify, $D^*$=knee $p$로 reweight | 해결됨(data-hungry; $D^*$를 knee에 고정) |
| ③ | Observation당 노이즈 있는 0/1 label 하나, 같은 observation 재등장 안 함 | 모듈이 함수 smoothness를 통해 *인접* observation들에 대해 평균; 넓은 obs 커버리지 필요 | Contact boundary(④) 근처를 제외하면 작동 — [action-quantization.md](action-quantization.md)의 표준 sub-problem 1로 추적 |
| ④ | Credit assignment / compositionality — 여러 chunk가 co-quantize될 때 성공은 joint의 속성 | ②가 "몇 개"는 처리; "구체적으로 어느 파트너"는 잔차로 남음. Clean fix = paired counterfactual(비용 큼, fallback만) | **OPEN** — [label-module-architecture.md](label-module-architecture.md)의 표준 sub-problem 2로 추적 |

## Experiment Log

- **2026-07-09** — 타임아웃-실패가 horizon-confound artifact가 아니라 진짜 회복불가 실패임을 확인: coarse 실행의 step 절약이, fine 실행 하에서는 성공했겠지만 타임아웃된 episode를 구제할 수 없음 — 그런 episode가 발견되지 않음. 이는 horizon 조정 없이도 binary success가 valid label임을 검증한다. 잔여 위생 항목: 분석에서 raw $m$ count 대신 $m/L$ 또는 per-chunk 비율을 쓸 것.

## Decision Status

Estimand/label 정의($\theta_i$, forward이지 posterior가 아님)와 $m$-stratify + $D^*$-at-knee marginalization 접근법은 `decided`. Hazard ④(credit assignment)는 여전히 open이므로 `exploring`.
