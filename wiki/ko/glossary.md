---
type: glossary
title: 표기법 용어집
date: 2026-07-10
tags: [glossary]
twin: ../en/glossary.md
last_synced: 2026-07-10
---

# 표기법 용어집

`wiki/en/`과 `wiki/ko/` 전체에서 사용하는 표기법의 공유 기준 문서. 여기 나열된 기호는 한글판에서도 **번역하지 않고 그대로** 유지한다 — `korean-sync`는 논문/방법론 노트를 번역하기 전에 반드시 이 파일을 참조해, 언어 간·논문 간 표기 일관성을 유지해야 한다. 새로 수집한 논문에서 새 기호가 등장하면 이 파일에 먼저 추가한다.

`raw/proposal.md` §11에서 시드됨.

| 기호/용어 | 의미 | 어디서 쓰나 |
|---|---|---|
| $\theta_i$ | forward estimand $P(\text{success} \mid \text{quantize chunk } i, \text{obs}_i)$ — **학습 label** | 데이터/label 쪽 |
| posterior | $P(\text{bit}=1 \mid \text{obs}, \text{success})$ — 수집 확률 $p$가 섞여 있어 label로 쓸 수 없음, 진단용으로만 사용 | signal 존재 확인 |
| $p$ | mixed-quant rollout 수집 시 chunk별 Bernoulli 압축 확률 | rollout 수집 |
| $D^*$ | 배포 $m$-분포. knee $p^*$에 고정해 label을 배포 operating point에 묶는다 | label marginalize |
| $m / L$ | 한 episode에서 co-quantize된 chunk 수 / 전체 chunk 수. $m \sim \text{Binomial}(L, p)$ | stratify |
| $\kappa$ (kappa) | **추론 시 라우팅 threshold**(operating point). label이 아니라 배포/eval 문턱 | 라우터 배포·sweep |
| $q$ | 압축비 $H/T$. v1 coarse는 $q=4$(`m4`), fallback은 $q=2$(`m8`) | decoder 선택 |
| contrast | $\sigma(\theta_{\text{fine}}) - \sigma(\theta_{\text{coarse}})$, 모듈의 chunk별 coarse-비용 점수 | 라우팅 판단 |

> **$p$/$D^*$와 $\kappa$를 혼동하지 말 것:** $p$/$D^*$는 데이터 수집·label operating point를 결정하고, $\kappa$는 추론 시 라우팅 문턱이다. 서로 다른 축이다.

## 도메인/프로젝트 용어 (번역하지 않고 고유명사·약어 그대로 유지)

GR00T N1.5, EagleBackbone, FlowmatchingActionHead, RoboCasa(-Kitchen), LIBERO, DexJoCo, Kinetix, ATQ (Adaptive Temporal Quantization), FAST, DCT+BPE.
