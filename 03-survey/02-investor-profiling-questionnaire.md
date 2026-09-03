# 온보딩 투자 성향 설문 — 확정 명세 v1.1

> **기준선 변경 (2026-09-03)** — 프론트엔드 저장소에 **10문항 draft가 이미 구현**되어 있다
> (`frontend/src/data/onboardingQuestions.ts`, `questionnaire-safe-t-v2-frontend-draft`).
> 미팅이 요구한 *"질문 / N개 선택지 / 세부 정보 명세"* 3요소를 모두 갖추고 있고 백엔드 ID 규약도 따르므로,
> **이 문서는 프론트엔드 draft를 확정안으로 채택**하고 거기에 없는 **① 시나리오 매칭 쓰임 ② 채점 기준선 연결 ③ 백엔드 반영 과제**를 더한다.
>
> | 저장소 | 상태 | 소관 |
> |---|---|---|
> | `frontend/src/data/onboardingQuestions.ts` | **10문항** · `questionnaire-safe-t-v2-frontend-draft` | 프론트엔드 담당 |
> | `backend/fixtures/questionnaire.json` | **7문항** · `questionnaire-safe-t-v1` | 엔진 담당 |
> | **이 문서** | **확정 명세 · 채점 연결 정의** | **기획 담당** |
>
> ⚠️ backend·frontend 저장소는 기획 담당의 관리 범위 밖이다. 아래 §3의 차이는 **지시가 아니라 전달·확인 사항**이다.

---

## 0. 설계 원칙

| # | 원칙 |
|---|---|
| **P1** | 모든 문항은 **시나리오 매칭** 또는 **채점 기준선** 중 하나 이상에 쓰인다 |
| **P2** | 1화면 1문항. 각 선택지에 **`detail` 보조 설명**을 붙여 용어 장벽을 낮춘다 |
| **P3** | 전 문항 **필수**. 뒤로 가기 허용 |
| **P4** | 실명·계좌·실제 자산 규모를 묻지 않는다 *(스펙 비기능 조건)* |
| **P5** | **엔진은 응답을 해석하지 않는다.** 원본 응답만 append-only 저장하고, 성향 해석·시나리오 추천은 Selection Orchestrator(LLM)가 수행한다 *(`backend-engine-spec.md`)* |

---

## 1. 확정 문항 10개

> `prompt`·`detail`·선택지 문구는 **프론트엔드 구현이 정본**이다. 아래는 대조 및 채점 연결용 요약이다.

| # | questionId | 카테고리 | 질문 | 유형 | 선택 |
|---|---|---|---|---|---|
| 1 | `survey-q-experience` | 투자 경험 | 투자 상품을 직접 판단해 본 기간은? | SINGLE | 1 |
| 2 | `survey-q-risk-preference` | 위험 선호 | 불확실성이 큰 상황에서 선호하는 판단 방식은? | SINGLE | 1 |
| 3 | `survey-q-leverage` | 상품 이해 | 레버리지·인버스의 일간 수익 구조를 어느 정도 이해하나? | SINGLE | 1 |
| 4 | `survey-q-loss-response` | 손실 대응 | 예상과 반대로 급격히 움직일 때 반응은? | SINGLE | 1 |
| 5 | `survey-q-source-check` | 정보 검증 | 커뮤니티 정보를 공시·신뢰 보도로 교차 확인하나? | SINGLE | 1 |
| 6 | `survey-q-interests` | 관심 영역 | 관심 있는 자산·산업은? | MULTI | 1–3 |
| 7 | `survey-q-assessment-risk` | 평가 난이도 | 이번 평가에서 감수할 수 있는 변동 수준은? | SINGLE | 1 |
| 8 | `survey-q-holding-period` | 투자 기간 | 판단 시 주로 생각하는 보유 기간은? | SINGLE | 1 |
| 9 | `survey-q-evidence-priority` | 판단 기준 | 가격 방향 판단 시 가장 먼저 확인하는 근거는? | SINGLE | 1 |
| 10 | `survey-q-learning-goal` | 학습 목표 | 이번 평가에서 확인하고 싶은 역량은? | MULTI | 1–2 |

---

## 2. 문항별 쓰임 — 시나리오 매칭과 채점 연결

> **이 표가 이 문서의 핵심 기여다.** 프론트엔드 draft에는 문항과 선택지만 있고, 각 응답이 무엇에 쓰이는지는 정의돼 있지 않다.

| # | questionId | 선택지 | → 시나리오 매칭 | → 채점 기준선 |
|---|---|---|---|---|
| 1 | `experience` | none / under-one / one-three / over-three | **난이도**: 정보 밀도, 상충 정보의 양 | — |
| 2 | `risk-preference` | preserve / balanced / opportunity | **가격 변동 강도** | 「판단의 일관성」 참조값 |
| 3 | `leverage` | unfamiliar / basic / confident | **레버리지·인버스 자산 포함 여부와 비중** | 「레버리지 통제」(20) 기준선 · **선언한 이해도 vs 실제 판단의 괴리** |
| 4 | `loss-response` | immediate / review / hold | **역방향 급변 구간 포함 여부** | 「손실 이후 행동」(10) 기준선 |
| 5 | `source-check` | true / false | **커뮤니티 게시물 밀도** | 「정보 출처 검증」(15) 기준선 — **선언한 습관 vs 실제 열람 로그 대조** ⭐ |
| 6 | `interests` | growth / defensive / bond / leverage / macro | **가명 자산의 유형 프로파일** | — |
| 7 | `assessment-risk` | low / medium / high | **시나리오 변동성 등급 선택** | 🔴 「위험 한도 준수」(20) — **아래 §4 참조** |
| 8 | `holding-period` | intraday / short / medium / long | 고정 horizon(시장시간 180분)과의 **기대 시계 정합** | 「판단의 일관성」 — 짧은 horizon에 장기 관점을 적용했는지 |
| 9 | `evidence-priority` | price / fundamental / flow / cross-check | 시나리오의 **정보 유형 구성**(차트 주도 vs 공시 주도) | 「판단 근거의 구체성」(5) — **선언한 우선 근거 vs 실제 `reasonTags` 대조** ⭐ |
| 10 | `learning-goal` | source / volatility / bias / record | 목표에 맞는 시나리오 우선 배정 | 결과 보고서의 **강조 항목** 결정 |

### ⭐ 대조가 만드는 진단 문장

| 근거 | 보고서 문장 |
|---|---|
| Q5 × 열람 로그 | *"커뮤니티 정보를 교차 확인한다고 답하셨습니다. 이번 평가에서 커뮤니티 게시물을 본 뒤 30초 이내에 판단한 것이 6회, 공시를 열어본 것은 1회였습니다."* |
| Q9 × `reasonTags` | *"판단 시 '실적과 공시'를 먼저 본다고 하셨지만, 실제 기록한 근거 태그는 9건 중 7건이 `NEWS`와 `INTUITION`이었습니다."* |
| Q3 × 레버리지 자산 판단 | *"레버리지의 '기본 개념을 안다'고 하셨습니다. 그런데 기초자산이 제자리로 돌아온 구간에서 레버리지 자산에 UP을 유지하셨습니다."* |

---

## 3. 참고 — 구현 저장소와의 차이 (전달 사항)

> 아래는 문서 작성 과정에서 대조하며 확인된 차이다. **조치는 각 저장소 담당의 판단 사항**이며,
> 여기서는 *"기획 문서 기준으로는 이렇게 정의돼 있다"*를 남기는 것이 목적이다.

`backend/fixtures/questionnaire.json`은 확인 시점 기준 **7문항 v1**이었다.

### 3.1 문항 수 차이
프론트엔드 draft에만 있는 3문항 — `survey-q-holding-period` · `survey-q-evidence-priority` · `survey-q-learning-goal`

### 3.2 필드 차이 — `category` · `detail`

| 필드 | 프론트엔드 | 백엔드 fixture | 문제 |
|---|---|---|---|
| `category` | ✅ 있음 (투자 경험, 위험 선호 …) | ❌ 없음 | 화면 그룹 라벨 |
| `detail` | ✅ 문항·선택지 **모두** 있음 | ❌ 없음 | **선택지 보조 설명이 사라진다.** 용어 장벽을 낮추는 핵심 장치 |

> **이대로면 프론트가 문구를 자체 보유하게 되어 백엔드 문항과 어긋난다.**
> 설문 문구는 **버전 고정 대상**(`questionnaireVersionId`)인데 두 곳에 나뉘면 버전 관리가 깨진다.
> → 문구를 한 곳에서 관리하려면 fixture 쪽에도 `category`·`detail`이 필요해 보인다. **판단은 엔진 담당 소관.**

### 3.3 타입 차이
`survey-q-source-check`가 백엔드는 `BOOLEAN`, 프론트는 `SINGLE_CHOICE`(예/아니요)다. 문서상으로는 선택지에 `detail`을 붙일 수 있는 **`SINGLE_CHOICE` 기준으로 기술**한다.

### 3.4 버전 ID
프론트 draft는 `questionnaire-safe-t-v2-frontend-draft`다. 확정 시 **`questionnaire-safe-t-v2`**로 정리하자는 제안을 9/6 미팅 안건으로 올려둔다.

---

## 4. 🔴 미해결 — 「위험 한도 준수」(20점) 측정 방식

**문제.** 스펙 배점표에서 최대 배점인 「위험 한도 준수」는 *"사전에 밝힌 위험 수준과 실제 판단의 일치 여부"*로 정의된다. 그러나 `survey-q-assessment-risk`는 **낮음/보통/높음 정성 3단계**이고, 백엔드 엔진은 규범적으로 다음을 **생성·추론하지 않는다.**

> 주문, 보유량, 비중, 현금, 노출, 손익, MDD, 유효 레버리지, **집중도** — `backend-engine-spec.md`

즉 **"한도를 얼마나 넘었는가"를 계산할 정량 입력이 엔진에 없다.** 「자산 집중도」(10점)도 item당 자산이 하나면 개념 자체가 성립하지 않는다. **합계 30점의 측정 방식을 재정의해야 한다.**

**어디서 정하는가.** `evaluationPlanRef`가 가리키는 **외부 승인 rubric**이 *"각 평가 항목의 공개 정보·행동·시장 checkpoint와 증거 부재 처리 규칙"*을 정의하도록 되어 있다. 이 rubric은 아직 작성되지 않았다.

**선택지**

| 안 | 「위험 한도 준수」 재정의 | 「자산 집중도」 |
|---|---|---|
| **가** | 선언한 변동 수준(low/medium/high) 대비 **고변동 구간에서의 판단 행태**를 LLM이 정성 판정 | 폐지, 배점을 다른 항목으로 이동 |
| **나** | 온보딩에 **정량 손실 한도 문항을 추가**(11번째) | 동일 |
| **다** | 항목명을 **「위험 인식 일관성」**으로 바꾸고, 선언 수준과 `reasonTags`·판단 변경 패턴의 정합으로 정의 | 「판단의 일관성」에 흡수 |

> **제안: 다.** 포지션이 없는 PoC 구조에 가장 정직하게 맞고, 문항 추가 없이 기존 10문항으로 채점된다.
> 어느 안이든 **rubric 작성이 선행**되어야 하며, 이는 기능명세서 4번(AI 및 데이터 처리 방식)에 기재할 내용이다.

---

## 5. 후속

**기획 담당 (이 문서 범위)**
- [ ] 9/6 미팅에서 §4(배점 30점 재정의) 결론을 받아 이 문서와 기획서에 반영
- [ ] 확정 문항·채점 연결을 **기능명세서 4번(AI 및 데이터 처리 방식)**에 옮겨 기술

**팀에 전달할 사항** *(각 담당 소관)*
- 문항 수·필드·타입·버전 ID 차이 (§3)
- Selection Orchestrator의 시나리오 선택 로직에 §2 매핑이 반영되는지
- `evaluationPlanRef`가 가리킬 **rubric 작성 주체와 일정** (§4의 선행 과제)
- 심사위원용 프리셋 프로파일 (설문 스킵 경로) — 기능명세서 5번 기재 대상
