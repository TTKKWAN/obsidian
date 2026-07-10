---
type: tags
title: 태그 택소노미
date: 2026-07-10
tags: [meta]
twin: ../en/tags.md
last_synced: 2026-07-10
---

# 태그 택소노미

`wiki/en/`(및 미러링된 `wiki/ko/`) 전체 파일 frontmatter에서 쓰는 태그의 단일 기준 문서. **새 태그는 반드시 여기 먼저 추가한 뒤에 frontmatter에서 사용해야 한다.** `data-collector`는 `(pending review)` 표시를 붙여 태그를 잠정적으로 추가할 수 있으나, 이를 확정(승인/반려)하는 것은 세션 종료 시 `wiki-architect`뿐이다.

## 도메인/환경 태그

- `simulation`
- `real-robot`
- `dataset`
- `web`
- `mobile`
- `macos`, `ubuntu`, `windows` (이 로보틱스 전용 프로젝트에는 아직 해당 없지만, 프로젝트 관례상 택소노미에 유지)

## 연구 태그 (action-quantization 프로젝트)

- `vla` — vision-language-action 모델 전반
- `action-quantization` — 이 프로젝트 핵심 주제의 상위 태그
- `temporal-compression` — 시간축(horizon) 압축 특정 (weight/activation/vision quantization과 구분)
- `tokenization`
- `discretization`
- `flow-matching`
- `diffusion-policy`
- `chunking`
- `action-chunk`
- `router` — dispatch/라우팅 모듈 (예: quantization signal router)
- `success-grounded-labeling` — reconstruction proxy가 아니라 실제 rollout success로 grounding된 label
- `mixed-quant-rollout`
- `credit-assignment` — co-quantize된 chunk들 간 joint success 귀속(compositionality) 문제
- `groot` — GR00T N1.5 backbone
- `robocasa`
- `libero`
- `dexjoco`
- `kinetix`
- `atq` — Adaptive Temporal Quantization, 이 프로젝트가 확장하는 reference paper
- `test-time-compute`
- `execution-horizon`

## 거버넌스 규칙

여기 등재되지 않은 태그는 어떤 파일의 frontmatter에도 쓸 수 없다(먼저 여기 추가하거나, `data-collector`가 `(pending review)`로 표시하고 `wiki-architect`가 확정해야 함 — `agents/wiki-architect.md` 참고).
