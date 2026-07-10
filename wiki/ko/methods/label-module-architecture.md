---
type: method
title: "Method: Label / Module Architecture"
date: 2026-07-10
tags: [action-quantization, credit-assignment, groot]
twin: ../../en/methods/label-module-architecture.md
last_synced: 2026-07-10
---

# Method: Label / Module Architecture

표기법과 GR00T/decoder 명명법은 [용어집](../glossary.md) 참고.

## Current Approach

**Backbone:** GR00T N1.5, frozen EagleBackbone + FlowmatchingActionHead. **Benchmark:** RoboCasa-Kitchen(primary), DexJoCo, Kinetix(LIBERO도 scope에서 언급됨). **Collection 체크포인트:** `groot_n1_5_bs64_moe_pyramid_K3_raw16_m8_m4_b_only_no_metaq_no_balance/checkpoint-60000`(`GR00T_N1_5_FairMoe`).

학습된 decoder: fine `action_decoder`($T=16$), `m8_action_decoder`($T=8$), `m4_full_action_decoder`($T=4$). 체크포인트 내장 ATQ router는 우회되고, mixed-quant rollout 수집 중 head가 직접 강제된다([dataset-labeling.md](dataset-labeling.md) 참고).

**이름 함정 (진짜 pitfall이라 그대로 남김):** 사용자 표기 "m2"(압축 factor 2, $H16 \to T8$)는 code `m8_action_decoder`(출력 $T=8$로 명명됨)에 대응한다. Code `m2_action_decoder`($T=2$)는 다른, 더 harsh한, 미사용 decoder다. **v1 coarse = $q4$ = `m4_full_action_decoder`.** 현재는 binary(main vs. coarse decoder 하나); 3-way 라우팅(main/m8/m4)은 의도된 later extension이며 아직 구현되지 않았다.

**환경 제약:** policy server(`gr00t` conda env)와 sim client(`robocasa_gr00t` conda env)는 zmq로 통신하며 프로세스를 공유할 수 없다. Masked-rollout collection은 모두 client에서 실행된다.

**Extend-don't-modify 경계:** GR00T N1.5, RoboCasa-Kitchen, LIBERO 코드베이스는 여기서 참조·문서화만 한다. 이 위키(및 그 에이전트)의 어떤 워크플로우도 해당 코드베이스를 직접 수정하지 않는다 — 이들에 대한 아키텍처 결정은 문서로만 기록되며, 실제 코드 변경은 여기가 아니라 해당 프로젝트 자체 저장소에서 이루어진다.

## Alternatives Considered

- **지금 당장 3-way 라우팅(main/m8/m4)** — 보류; binary 라우팅(main vs. coarse 하나)이 현재 scope이며, 3-way는 binary 라우팅이 검증된 이후의 의도된 later extension이다.
- **GR00T head/router 직접 수정** — 기각; extend-don't-modify를 위반한다. Client-side-only 배포([module-training.md](module-training.md) 참고)가 바로 이를 피하기 위해 선택된 경로다.

## Open Sub-problems

**표준 sub-problem 2 (항상 최신 상태로 유지): quantization-safety label의 인접 state-space 영역에 대한 일반화 가능성 — credit assignment / compositionality(hazard ④).** Episode 내 여러 chunk가 함께 co-quantize될 때, episode 성공은 *joint* 압축 chunk 집합의 속성이지 어느 하나의 chunk에 깔끔하게 귀속시킬 수 없다. $m$-stratification([dataset-labeling.md](dataset-labeling.md), hazard ② 참고)은 "몇 개의 chunk가 co-quantize됐나"는 해결하지만 "구체적으로 어느 파트너 조합이 중요했나"는 잔차 분산으로 남긴다. Clean fix는 **paired counterfactual**이다: on-policy mixed-quant snapshot에서 chunk $i$ 하나만 분기해(fine vs. coarse, downstream은 동일하게 유지) 다른 모든 것을 고정한 채 $\theta_i$를 per-chunk로 측정하는 것. 이는 rollout 비용이 2배이므로 **fallback**이다 — (수집되고 나면) `m4`-aggregate 데이터가 contact-rich chunk 근처에서 flat하거나 신뢰할 수 없는 것으로 판명될 때만 트리거되며, 기본 접근법은 아니다.

폐기된 방식과의 중요한 구분: paired-counterfactual의 snapshot/restore *엔지니어링*은 폐기된 old clean-fine counterfactual 방식에서 재사용 가능하다 — 다른 점은 observation 분포뿐이다(여기서는 on-policy mixed-quant, 그쪽은 off-distribution clean-fine — 그래서 그 방식이 폐기됨; [action-quantization.md](action-quantization.md)의 Alternatives Considered 참고).

## Experiment Log

_(아직 없음 — 이 파일은 아키텍처/credit-assignment 결정을 구체적으로 추적한다; 이미 로그된 `m8`/`m4` 실험 항목은 [dataset-labeling.md](dataset-labeling.md)와 [module-training.md](module-training.md) 참고)_

## Decision Status

`exploring` — paired counterfactual은 아직 필요하지도 구현되지도 않은 fallback으로 남아있다; contact-boundary chunk 근처에서 `m4` aggregate 데이터의 flatness 여부를 확인한 후 `decided`(채택) 또는 `exploring` 유지(그것 없이도 충분함)로 이동한다.
