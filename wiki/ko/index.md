---
type: index
title: 색인
date: 2026-07-10
tags: [meta]
twin: ../en/index.md
last_synced: 2026-07-10
---

# 색인

위키 카탈로그. `wiki-architect`가 관리하며(`agents/wiki-architect.md` 참고), `wiki/`를 건드리는 모든 세션 종료 시 갱신한다.

## 프로젝트 초점

이 vault는 VLA action chunk에 대한 **success-grounded quantization signal routing** 연구를 기록한다 — canonical spec은 [raw/proposal.md](../../raw/proposal.md), 계속 발전하는 연구 방향과 표준 미해결 질문은 [research-directions.md](research-directions.md) 참고.

## 논문

| Slug | 제목 | 상태 | 태그 | 링크 |
|---|---|---|---|---|
| `atq-adaptive-temporal-quantization` | *(잠정 — ATQ reference paper "Adaptive Temporal Quantization"으로 추정; ingest 시 확인)* | backlog | `atq`, `action-quantization` | [raw/papers/atq-adaptive-temporal-quantization/original.pdf](../../raw/papers/atq-adaptive-temporal-quantization/original.pdf) |
| `atq-supplementary` | *(잠정 — 위 ATQ 논문의 supplementary material로 추정; ingest 시 확인)* | backlog | `atq` | [raw/papers/atq-supplementary/original.pdf](../../raw/papers/atq-supplementary/original.pdf) |
| `dynamic-test-time-compute-control-policy` | *(잠정, 파일명 "Dynamic Test-Time Compute Scaling in Control Policy"에서)* | backlog | `test-time-compute` | [raw/papers/dynamic-test-time-compute-control-policy/original.pdf](../../raw/papers/dynamic-test-time-compute-control-policy/original.pdf) |
| `moh` | *(잠정, 파일명 "MoH"에서)* | backlog | _(ingest 시 결정)_ | [raw/papers/moh/original.pdf](../../raw/papers/moh/original.pdf) |
| `aac` | *(잠정, 파일명 "AAC"에서)* | backlog | _(ingest 시 결정)_ | [raw/papers/aac/original.pdf](../../raw/papers/aac/original.pdf) |
| `dynamic-execution-horizon-prediction` | *(잠정, 파일명 "Dynamic Execution Horizon Prediction"에서)* | backlog | `execution-horizon` | [raw/papers/dynamic-execution-horizon-prediction/original.pdf](../../raw/papers/dynamic-execution-horizon-prediction/original.pdf) |
| `vlash` | *(잠정, 파일명 "vlash"에서)* | backlog | `vla` | [raw/papers/vlash/original.pdf](../../raw/papers/vlash/original.pdf) |

7개 slug 모두 `data-collector`가 실제로 논문을 열어 ingest하기 전까지는 잠정 상태다(`agents/data-collector.md` 참고); 이때 제목/태그가 확정된다.

## Methods

| 파일 | Decision Status |
|---|---|
| [action-quantization.md](methods/action-quantization.md) | exploring |
| [dataset-labeling.md](methods/dataset-labeling.md) | exploring |
| [module-training.md](methods/module-training.md) | exploring |
| [label-module-architecture.md](methods/label-module-architecture.md) | exploring |

## 연구 방향

[research-directions.md](research-directions.md) 참고. 항상 최신 상태로 유지되는 표준 미해결 sub-problem:
1. Single-rollout success estimate의 통계적 신뢰도.
2. Quantization-safety label의 인접 state-space 영역에 대한 일반화 가능성.

## 용어집

표기법($\theta$, $\kappa$, $p$, $D^*$, $m/L$, $q$, ...)은 [glossary.md](glossary.md) 참고.

## 태그

전체 택소노미와 거버넌스 규칙은 [tags.md](tags.md) 참고.
