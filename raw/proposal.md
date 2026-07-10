# Proposal: Success-Grounded Quantization Signal Router

> **이 문서의 위치.** 프로젝트를 처음 보는 사람(또는 LLM)이 이 한 파일만 읽고 "무엇을, 왜, 어떻게"를
> 이해하도록 만든 canonical proposal이다. 개념의 상세 척추는 `docs/QUANTIZATION_SIGNAL_CORE_IDEA.md`,
> estimand/label 규정은 `.claude/CLAUDE.md`(§ Estimand & label)

---

## TL;DR (한 줄 요약)

action chunk를 실행하기 **직전**, chunk 경계의 obs만 보고 **"이 chunk를 coarse(=압축된 짧은
decoder)로 실행해도 task 성공에 지장이 없는가?"**를 판단하는 **per-chunk 모듈**을 만든다. 추론 시엔
chunk마다 fine/coarse decoder를 dispatch하는 **router**로 동작한다. 신규성은 **학습 신호**에 있다:
reconstruction-fit proxy(ATQ)가 아니라 **실제 mixed-quant rollout의 success로 grounding한 label**로
학습한다. 최종 산출물은 이 학습된 모듈이며, **라우터로 배포해 ATQ 자신의 라우터를 success–speed
tradeoff에서 이기는 것**으로 평가한다.

---

## 1. 문제와 동기

GR00T 류 VLA policy는 action을 **chunk 단위**로 뽑는다(한 번 forward에 H=16 step). chunk를 더 적은,
더 긴 step으로 **시간축 압축**하면(T<H, 압축비 q=H/T) 같은 물리적 결과를 더 적은 decoder 호출로 낼 수
있어 **빠르다**. 하지만 모든 chunk가 압축에 안전한 건 아니다 — contact-rich·정밀 구간을 압축하면 task가
실패한다.

**ATQ (Adaptive Temporal Quantization)**는 이 dispatch를 학습하는 참조 연구다. 그러나 ATQ의 라우터는
각 decoder의 **reconstruction fit**(coarse 출력이 fine 출력을 얼마나 잘 재현하나)이라는 *implicit proxy*로
학습된다. 재현이 잘 된다 ≠ task가 성공한다. ATQ 논문의 Limitation 절 자체가 **"실제 rollout success로
grounding한 quantizability supervision"**을 future work으로 제안한다.

**이 프로젝트가 그 future work을 실현한다.** decoder들은 ATQ-style로 학습된 것을 그대로 재사용하고,
바꾸는 것은 오직 **라우터의 학습 신호**다: reconstruction-fit → **success-grounded label**.

### 무엇이 아닌가 (scope 방어)
- ATQ **재현이 아니다**. ATQ는 동기를 주는 reference paper. "라우터가 없다"는 뜻도 아니다 — 우리도
  라우터를 배포한다. 다른 건 **학습 신호 하나**뿐.
- action-token quantization(FAST/DCT+BPE), weight/activation precision quant(int8/int4), vision-input
  압축 — **전부 아니다**. 여기서 "quantization"은 오직 **temporal-horizon 압축**이다.
- 폐기된 두 방식(clean-fine counterfactual, post-hoc block-aggregation)을 재도입하지 않는다(§9).

---

## 2. 핵심 정의

### Quantization Signal
chunk 경계에서 **실행 전에 한 번**, 현재 obs만으로 내리는 per-chunk 결정:
- "이 chunk 전체를 coarse로 보내도 task 성공에 지장이 없는가?"
- 출력: 연속 confidence(0–1), 필요 시 threshold로 binary.
- 판단 기준: 압축이 **task success**에 영향을 주는가 — 단순 "free-space인가"가 아니다.

### "Coarse 실행"의 정확한 의미
= **학습된 coarse decoder를 호출**하는 것(v1: `m4_full_action_decoder`, q=4). block-aggregation(delta
차원은 window 합산, absolute/discrete 차원은 마지막 값 latch)은 그 coarse decoder의 **학습 target일
뿐**, rollout에서 coarse를 만드는 방법이 절대 아니다.

---

## 3. Estimand & Label (핵심 — 정확히 읽을 것)

모듈이 예측하는 것은 개입(intervention) 하의 **forward** 성공 확률이다:

    θ_i = P(success | chunk i를 quantize, obs_i)

**posterior `P(quantized | obs_i, success)`가 아니다.** 둘은 Bayes로 엮이지만 다르다: posterior엔
수집률 p가 곱해져 있어(`P(bit=1|obs,succ) = θ_i·p / P(succ|obs)`) **데이터를 어떤 p로 모았느냐에 따라
값이 흔들린다**. forward θ_i는 p를 벗겨낸 clean estimand다. → posterior는 **label이 아니라 "signal
존재 여부 진단용"**으로만 쓴다.

### co-quantization 의존성과 marginalization
한 chunk의 성공은 같은 episode에서 **다른 몇 개의 chunk가 같이 압축됐나(m)**에 달려 있다 → θ_i(m)은
m에 대한 곡선이다. 각 chunk가 i.i.d. Bernoulli(p)로 압축되므로 **m ~ Binomial(L, p)**. 곡선을 배포
분포 D*로 marginalize해 하나의 label로 접는다:

    θ_i = Σ_m P(m | D*) · θ_i(m)

**D*를 고르는 것 = operating 압축률 p를 고르는 것.** D*를 **knee p***(success–speed tradeoff가 가장
좋은 지점)에 고정해 label을 배포 operating point에 묶는다 — 데이터가 우연히 모인 p에 끌려다니지 않게.

---

## 4. 파이프라인

1. **Mixed-quant rollout.** GR00T closed-loop 실행 중 chunk마다 i.i.d. Bernoulli(p) 동전으로 **head를
   강제**한다: bit=0 → fine(main, T=16), bit=1 → coarse(학습된 coarse decoder 호출, v1 `m4`).
   체크포인트 내장 ATQ router는 **우회**. 따라서 경계 obs는 **on-policy mixed-quant** 분포에서 나온다
   (clean fine episode가 아님).
2. **Success filter.** 성공 episode만 남긴다. 그 per-chunk `(obs, bit, m)`가 데이터가 된다. **실패
   rollout은 버린다 — 절대 학습 데이터로 쓰지 않는다.** obs는 오직 success로 grounding된다.
3. **Negative mechanism (success-only인데도 signal이 실리는 이유).** *위험한* chunk가 압축되면 그
   episode는 fail → 걸러진다. 그래서 살아남은 success들 사이에서 위험한 chunk는 대부분 fine으로 남고,
   진짜 coarse-able한 chunk만 자유롭게 압축된 채 나타난다 → success set에서의 **부재가 negative를
   만든다**. ⚠️ 이건 **압축 강도가 충분히 셀 때만 발화**한다(§6).
4. **Label = θ_i** (§3): 각 chunk 결과를 m으로 stratify → D*=knee p*로 reweight.
5. **학습 + 배포.** 작은 모듈 obs→θ 학습 → **라우터로 배포** → success–speed를 baseline과 비교 채점
   (§7).

---

## 5. Label 통계의 hazard와 처리

| # | hazard | 처리 | 상태 |
|---|---|---|---|
| ① | chunk가 fine으로 남음 = "안 눌러봄"이지 "압축 불가"가 아님 | success filter가 여러 mask를 aggregate; 위험-chunk-압축 rollout은 fail→drop → chunk는 대부분 fine으로 생존 → θ_i 낮음 | **강도 충분할 때만** 작동. RoboCasa×q2는 너무 약해 flat → 그래서 m4 |
| ② | 성공이 co-quant m에 의존 → naive 평균은 수집 p에 끌려감 | m으로 stratify, D*=knee p로 reweight (m~Binomial(L,p)) | 해결(data-hungry; D*를 knee에 고정) |
| ③ | obs당 noisy 0/1 하나, 같은 obs 재등장 안 함 | 모듈이 *인접* obs로 평균(함수 smoothness); 넓은 obs 커버리지 필요 | contact boundary(④) 빼고 작동 |
| ④ | **credit assignment / compositionality (OPEN)**: 1&3을 같이 압축 → 성공은 *joint*의 성질 | ②가 "몇 개"는 처리; "구체적으로 어느 파트너"는 잔차. clean fix = **paired counterfactual** | OPEN; m4 aggregate도 flat이면 fallback |

---

## 6. 강도: 왜 m8은 flat이었고 왜 지금 m4인가

signal 존재와 올바른 estimand는 **독립인 두 축이고 둘 다 성립해야 한다.** 측정된 flat
posterior(`P(bit=1|success) ≈ p`)는 **"signal 없음"이 아니다** — ATQ 논문(Tables 1–3)이 non-quantizable
chunk의 존재를 증명한다: uniform 압축이 성공률을 붕괴시킴(real Q2 −29pp, DexJoCo Q2 −18pp, RoboCasa
Q3 −16pp).

실패 원인은 **강도**였다. **RoboCasa×q2(= code `m8`, factor 2)는 가장 약한 셀**(−5.6pp ≈ 우리 −7pp).
per-chunk 열화가 episode 실패 임계보다 작아 위험 chunk를 눌러도 episode가 거의 안 죽었고 → negative
mechanism 미발화 → flat.

**처방: v1 coarse = `m4_full_action_decoder` (q=4)** on RoboCasa + **낮은 mask p ≈ 0.1–0.2**(success
set을 populated하게 유지 + chunk 간 de-correlate → ②·④의 "몇 개" 완화). `m8`(q=2)은 m4가 성공률을
과도하게 붕괴시킬 때의 fallback.

---

## 7. 평가 프로토콜

학습된 모듈을 **체크포인트 내장 ATQ 라우터의 대체물**로 배포한다. 배포는 **client-side only** — server는
fine·coarse 두 branch와 ATQ router feature를 모두 반환하고, client가 모듈로 어느 branch를 쓸지 고른다
(head/server 수정 없음).

- **Routing 규칙:** `contrast = σ(θ_fine) − σ(θ_coarse)`; `bit = int(contrast ≤ kappa)`.
- **kappa = operating point:** kappa ↓ → 더 많이 coarse → 빠름(위험↑); kappa ↑ → 대부분 fine(안전·느림).
  **kappa를 sweep**해 success–speed 곡선을 그린다. coarse-rate {0.1,0.3,0.5,0.7,0.9} 분위수로 calibrate해
  모듈 간 곡선을 비교 가능하게 만든다.
- **Baselines:** all-fine(성공률 상한) · all-coarse(하한) · random-p(공짜 기준선) · **ATQ 자신의
  reconstruction-fit 라우터(이길 대상)**. 동일 dispatch 규칙으로 공정 비교.
- **승리 조건:** 동일 speedup에서 더 높은 성공률, 또는 동일 성공률에서 더 큰 speedup.

---

## 8. 현재 상태 (2026-07-09 기준)

**m8/q2 파이프라인은 end-to-end 검증 완료 — 단 예상대로 flat(negative control).**
- p={0.2,0.4,0.6,0.8}별 모듈 4개 학습 완료. θ_fine/θ_coarse AUC ~0.85–0.90(= task/phase 난이도
  baseline)이지만 **contrast_gap ≈ 0**(safe-to-quantize를 risky와 분리 못 함) — m8이 가장 약한 셀이라는
  전체 결론과 일치.
- 라우터 배포 client(`robocasa_service_qs_router.py`) + kappa calibration + eval sweep + summarize
  스크립트 모두 staged/submitted. 이 run은 **배포 파이프라인 검증 + negative control**이다: 4 모듈이
  flat이므로 QS-router 곡선은 random-p 선 위에 ≈ 앉을 것으로 예상.
- **진짜 signal 테스트는 m4/q4** — m4feat collection 진행 중(job `406006`). ATQ 논문이 여기선
  non-quantizable chunk가 실제로 존재한다고 예측.

**타임아웃-실패 = 진짜 회복불가 실패**로 확인됨 → horizon confound 해소(coarse의 step 절약이
would-solve-but-timed-out episode를 구제할 수 없음; 그런 episode 없음). binary success가 valid label,
horizon 변경 불필요. 잔여는 분석 위생뿐: raw count m 말고 m/L 또는 per-chunk를 쓸 것.

---

## 9. 열린 문제 & 폐기된 방식

**열린 knob — Credit assignment / compositionality (hazard ④).** m-stratify는 "몇 개"만 처리; "어느
파트너"는 잔차. clean fix는 **paired counterfactual**: on-policy mixed-quant snapshot에서 **chunk i만**
분기(fine branch vs coarse branch, downstream 동일)해 θ_i를 per-chunk로 깨끗이 측정. 비용 2×라서 default가
아닌 **fallback**(m4 aggregate도 flat일 때).

**폐기(재도입 금지):**
- **Old counterfactual (2026-07-03):** **clean fine episode**에서 한 chunk를 분기해 labeling. obs가
  fine-only 분포라 배포 off. (단 snapshot/restore *엔지니어링*은 재사용 가능 — 위 paired-counterfactual의
  메커니즘. 거기선 obs가 on-policy로 유지된다는 점만 다름.)
- **Post-hoc block-aggregation (2026-07-06):** fine decoder 16-step 출력을 block-sum해 "coarse"를 생성.
  라우터의 train/serve mismatch(잘못된 estimand). baseline ckpt에 coarse decoder가 없어 *강제됐던* 방식.

---

## 10. 환경 & 체크포인트

- **Backbone:** GR00T N1.5, flow-matching action expert. **Benchmarks:** RoboCasa-Kitchen(primary),
  DexJoCo, Kinetix. **HW:** RTX 5090.
- **Collection ckpt:** `groot_n1_5_bs64_moe_pyramid_K3_raw16_m8_m4_b_only_no_metaq_no_balance/checkpoint-60000`
  (`GR00T_N1_5_FairMoe`). 학습된 decoder: fine `action_decoder`(T16), `m8_action_decoder`(T8),
  `m4_full_action_decoder`(T4). 내장 ATQ 라우터는 우회하고 head를 강제.
- **이름 함정:** 사용자의 "m2"(압축 **factor** 2, H16→T8) = code `m8_action_decoder`(출력 T=8로 명명).
  code `m2_action_decoder`(T=2)는 다른·더 harsh·미사용 decoder. **v1 coarse = q4 = `m4_full_action_decoder`.**
  여전히 binary(main vs one coarse); 3-way(main/m8/m4)는 의도된 later extension.
- **Two-conda-env 제약:** policy server(`gr00t` env) ↔ sim client(`robocasa_gr00t` env)가 zmq로 통신,
  한 프로세스 공유 불가. masked-rollout collector는 모두 client다.

---

## 11. 용어집 (헷갈리기 쉬운 것)

| 기호/용어 | 의미 | 어디서 쓰나 |
|---|---|---|
| **θ_i** | forward estimand `P(success | quantize chunk i, obs_i)` — **학습 label** | 데이터/label 쪽 |
| **posterior** | `P(bit=1 | obs, success)` — p가 섞여 있어 label 아님, **진단용** | signal 존재 확인 |
| **p** | mixed-quant 수집 시 chunk별 Bernoulli 압축 확률 | rollout 수집 |
| **D\*** | 배포 m-분포. knee p*에 고정 → label을 배포 operating point에 묶음 | label marginalize |
| **m / L** | 한 episode에서 co-quantize된 chunk 수 / 전체 chunk 수. m~Binomial(L,p) | stratify |
| **kappa (κ)** | **추론 시 라우팅 threshold**(operating point). label 아님, 배포/eval 문턱 | 라우터 배포·sweep |
| **q** | 압축비 H/T. v1 coarse q=4(m4), fallback q=2(m8) | decoder 선택 |
| **contrast** | `σ(θ_fine) − σ(θ_coarse)`, 모듈의 chunk별 coarse-비용 점수 | 라우팅 판단 |

> **p/D\* vs kappa 혼동 주의:** `p`/`D*`는 **데이터 수집·label operating point**, `kappa`는 **추론 시
> 라우팅 문턱**. 서로 다른 축이다.
