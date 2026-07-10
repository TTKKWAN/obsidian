---
type: research-directions
title: 연구 방향
date: 2026-07-10
tags: [action-quantization, success-grounded-labeling]
twin: ../en/research-directions.md
last_synced: 2026-07-10
---

# 연구 방향

Append-only, 날짜 스탬프 방식. 새 항목은 날짜별 로그 맨 아래에 추가하며, 과거 항목은 절대 수정하지 않고 오직 이를 명시적으로 대체하는 새 항목만 추가한다. 표기법은 [용어집](glossary.md) 참고.

## 미해결 질문 (계속 갱신되는 목록)

이 두 항목은 append-only가 아니라 이해가 깊어질 때마다 제자리에서 갱신한다 — `raw/proposal.md` §5, §9에 따라 이 프로젝트가 항상 최신 상태로 유지해야 하는 두 개의 표준 open sub-problem이다:

1. **Single-rollout success estimate의 통계적 신뢰도.** 각 `(obs, bit, m)` 데이터 포인트는 노이즈가 섞인 0/1 rollout 결과 하나에서 나온다 — 같은 obs는 다시 등장하지 않는다. 현재 완화책(proposal.md §5, hazard ③): 모듈이 함수의 smoothness를 이용해 *인접* obs들에 대해 평균을 낸다고 가정. Open: 이 smoothing의 신뢰도를 어떻게 정량화/제한할 것인가, 특히 이 가정이 무너지는 것으로 알려진 contact boundary(④) 근처에서.
2. **Quantization-safety label의 인접 state-space 영역에 대한 일반화 가능성.** Hazard ④(credit assignment / compositionality)와 연결됨: 여러 chunk가 함께 co-quantize되면 성공은 *joint*의 속성이 되고, $m$-stratify는 "몇 개가 co-quantize됐나"만 해결할 뿐 "구체적으로 어느 파트너 조합이었나"는 잔차로 남는다. Clean fix(paired counterfactual, proposal.md §9)는 비용이 큰 fallback일 뿐 기본값이 아니다. Open: `m4` aggregate 데이터를 실제로 살펴봤을 때 $m$-stratify + smoothness만으로 충분한지, 아니면 paired counterfactual이 필요해지는지.

## 날짜별 로그

### 2026-07-10 — Vault 부트스트랩; proposal.md에서 시드

`raw/proposal.md`(프로젝트의 canonical spec)에서 직접 시드한 초기 항목, 위키 재설계 부트스트랩의 일부 (`.omc/plans/vla-wiki-vault-redesign.md` 참고).

- **프로젝트:** Success-Grounded Quantization Signal Router. Action chunk를 실행하기 직전, 이 chunk를 coarse(시간축 압축, $q=4$, `m4_full_action_decoder`) decoder로 보낼지 fine(`action_decoder`, $T=16$) decoder로 보낼지 결정하는 per-chunk 모듈. ATQ의 reconstruction-fit proxy가 아니라 실제 mixed-quant rollout success로 grounding된 label로 학습한다.
- **왜 관련있나:** ATQ(Adaptive Temporal Quantization)가 reference/inspiration paper이며, 그 논문 자체의 Limitations 절이 바로 이것(rollout-success-grounded quantizability supervision)을 future work으로 제안한다. 이 프로젝트가 그 future work을 실현한다 — ATQ-style decoder는 그대로 재사용하고, router의 학습 신호만 바꾼다.
- **현재 상태 (proposal.md §8 기준, 2026-07-09):** `m8`/`q2` 파이프라인이 negative control로서 end-to-end 검증 완료(예상대로 flat contrast_gap — RoboCasa×q2가 가장 약한 셀). 진짜 signal 테스트는 `m4`/`q4`(feature collection 진행 중, job `406006`).
- **Backlog:** 아직 ingest되지 않은 raw 논문 7편 — 상태는 [index.md](index.md) 참고. `Action_Quantization (12).pdf`는 ATQ reference paper 본문으로 추정(ingest 시 확인 필요); `Action_Quantization_supplementary_restore (2).pdf`는 그 supplementary material로 추정.

## Sync Log

_(append-only; `korean-sync`는 no-op 실행을 포함해 매 세션 정확히 한 줄을 여기에 남긴다 — `agents/korean-sync.md` 참고)_

- [2026-07-10] korean-sync: 8개 파일 동기화 완료 (bootstrap: index.md, tags.md, research-directions.md, glossary.md, methods/action-quantization.md, methods/dataset-labeling.md, methods/module-training.md, methods/label-module-architecture.md)
