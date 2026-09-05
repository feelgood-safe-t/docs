# Safe-T

## 프로젝트 정의

**Safe-T**는 실제 돈을 투자하기 전에 실제 시장을 바탕으로 만든 가명 시나리오를 경험하고,
사용자의 위험 대응과 의사결정 과정을 평가하는 AI 기반 투자 안전교육 플랫폼이다.

PoC는 사용자가 어떤 정보를 보고 어느 시점에 `UP` 또는 `DOWN` 의견을 냈으며 그 이유가
무엇인지 평가한다. 최종 수익이나 포트폴리오 성과를 평가하는 서비스가 아니다.

- 어떤 정보를 확인했는가
- 어느 시점에 방향을 판단했는가
- 판단 이유가 당시 공개 정보와 연결되는가
- 위험과 변동성 신호를 판단 근거와 확신도에 반영했는가
- 시간에 따른 판단·근거·확신도의 변화가 일관되는가

PoC의 `UP/DOWN`은 방향 의견이다. 주문·매수·매도·보유량·비중·현금·노출·포지션을
만들지 않으며, 의견을 암묵적인 “표준 가상 포지션”으로 변환하지 않는다.

평가를 통과하면 `INVEST PASS` 판정을 제공한다. 이는 플랫폼 자체 교육 평가 결과이며 공인
금융 자격이나 실제 투자 적격성을 뜻하지 않는다. 영속 인증 코드와 공개 검증 페이지는 운영형
WIP다.

---

## PoC 실행 프로파일

이번 PoC Backend Engine은 사용자·세션을 관리하지 않는 **stateless 변환 API**다.

| 구분 | PoC 계약 |
|---|---|
| 사용자 식별 | 없음. `participantId`와 사용자 계정이 없다. |
| 인증·인가 | 없음. token, credential, 소유자 검사가 없다. |
| 사용자 상태 | 프론트 메모리에서만 유지한다. |
| 서버 영속 저장 | 없음. 설문, 행동, 세션, Snapshot, 평가 결과를 DB에 저장하지 않는다. |
| 시장 진행 | 프론트가 문항별 180초 timer와 60배속 재생을 담당한다. |
| 데이터 전달 | 세 문항의 캔들·뉴스 전체를 한 번에 전달한다. |
| 평가 입력 | 최종 client bundle을 `safe-t-evaluation-snapshot/5.0`으로 변환한다. |
| LLM | 엔진에 포함하지 않는다. Snapshot을 받은 외부 Evaluation Service가 호출한다. |

질문지와 검수된 시나리오 catalog 같은 버전 고정 정적 자료는 서버가 읽을 수 있다. 이는 사용자
상태가 아니며 API 요청으로 생성·수정되지 않는다. 각 요청은 독립적이므로 호출자는 Snapshot을
만들 때 설문 응답과 세 문항의 최종 행동 묶음을 다시 보내야 한다.

공유하는 사용자 state·sequence·최근 요청 cache가 없으므로 정상적인 동시 접속에서도 각
브라우저의 응답이 섞이지 않는다. 동일 정적 release를 읽는 여러 worker도 같은 입력에
같은 Snapshot hash를 낸다. 다만 이는 기능적 격리이며, 공개 배포의 처리량·rate limit·TLS와
모니터링은 별도 운영 책임이다.

이 구조는 데모 통합 속도를 위한 의도적인 PoC 단순화다. 새로고침 복구, 서버가 보증하는 timer,
행동 원본성, 미래 데이터 차단과 사용자별 결과 조회를 제공하지 않는다.

---

## 제품 구조와 PoC 범위

| 모드 | 내용 | 상태 |
|---|---|---|
| 캐주얼 판단 시험 | 가명 시장 문항 세 개를 문항당 최대 3분씩 순차적으로 풀고 UP/DOWN과 이유를 기록 | **PoC 범위** |
| 포지션·거래 고도화 | 가상 자금, 주문, 체결, 보유량, 비중, 현금과 포트폴리오 위험 지표를 다루는 실전형 모드 | **WIP, PoC 제외** |

캐주얼 판단 시험은 실전형 모드의 축소 포트폴리오가 아니라 독립적인 의견 증거 기반 교육
평가다. 문항 사이에 포지션이나 자산 상태를 이월하지 않는다.

PoC는 다음 기능을 검증한다.

- 고정된 10문항 성향 설문과 원본 응답 검증
- 검수된 시나리오 세 문항 전달
- 프론트 메모리 기반 3분·60배속 재생
- 시점별 반복 방향 판단·필수 이유 태그·필수 확신도·뉴스 열람 기록
- 최종 client bundle의 결정론적 검증과 Evaluation Snapshot 생성
- 외부 LLM 기반 항목별 분석과 규칙 기반 합산·PASS 판정

포지션·거래 고도화는 화면에 `준비 중`으로 표시하며 현재 API, Snapshot, 점수와 PASS에 영향을
주지 않는다.

---

## 핵심 사용자 흐름

```text
질문지·검수된 시나리오 후보 조회와 성향 설문 작성
→ 외부 Selection LLM 또는 고정 규칙으로 세 문항 선택
→ 설문 응답으로 stateless assessment package 요청
→ 프론트에서 문항 1: 최대 3분 관찰·판단·완료
→ 프론트에서 문항 2: 최대 3분 관찰·판단·완료
→ 프론트에서 문항 3: 최대 3분 관찰·판단·완료
→ 설문 + 세 문항의 최종 이벤트 bundle 제출
→ Evaluation Snapshot v5와 snapshotHash 생성
→ 외부 LLM 항목별 분석 + 규칙 기반 점수·PASS
→ 결과 화면
```

프론트는 한 번에 현재 문항 하나만 보여준다. 전체 시험의 최대 표시 시간은 9분이며 문항 조기
완료에 따라 짧아질 수 있다. 서버는 세 시나리오 데이터를 이미 한 번에 전달했으므로 이 순서와
시간을 독립적으로 증명하지 않는다.

---

## PoC API

도메인 API는 다음 네 개뿐이다. 모두 인증 헤더 없이 호출한다.

| Method | Path | 역할 |
|---|---|---|
| `GET` | `/v1/poc/questionnaire` | 버전이 고정된 현재 10문항 질문지 반환 |
| `GET` | `/v1/poc/scenario-catalog` | Selection LLM용 검수 후보 10개의 ID·type·brief·version 반환 |
| `POST` | `/v1/poc/assessment` | 설문 검증 후 세 시나리오와 전체 캔들·뉴스 반환 |
| `POST` | `/v1/poc/snapshot` | 설문과 최종 행동 bundle 검증 후 Snapshot v5 생성 |

`/`, `/health`, `/ready`는 실행 상태 확인용이며 도메인 상태를 만들지 않는다. 참가자, 설문 제출,
세션 시작·조회, 건별 판단·열람 저장, 문항 완료, 시험 종료, Snapshot 목록·조회와 `/internal`
API는 없다.

### `GET /v1/poc/questionnaire`

`questionnaire-safe-t-v2`의 질문, 선택지, 질문 유형과 선택 범위를 반환한다. 질문지 자체는
버전이 고정된 읽기 전용 자료다.

### `GET /v1/poc/scenario-catalog`

외부 Selection LLM이 별도 수동 ID 목록 없이 후보를 선택하도록 version·checksum이
고정된 작은 catalog를 반환한다. 후보별 `scenarioId`, `scenarioVersionId`,
`scenarioChecksum`, `scenarioType`, `brief`, `sourceState`만 제공하고 캔들·뉴스는 넣지
않는다. 이 조회는 사용자 state를 만들지 않는다.

### `POST /v1/poc/assessment`

요청은 `questionnaireVersionId`와 10개 원본 `answers`를 포함한다. 엔진은 질문 ID, 선택지 ID,
단일·다중 선택 범위를 검증하지만 답을 성향 등급, 위험 한도 또는 투자 적합성으로 해석하지
않는다.

`scenarioIds`를 생략하면 검수 catalog에서 안정적으로 고정된 기본 세 문항을 사용한다. 외부
Selection Service는 선택 catalog에서 고른 정확히 세 개의 ID를 넘길 수도 있다. 이 선택은 교육용 문항
구성이며 자산 추천이 아니다.

응답은 다음을 포함한다.

- `safe-t-stateless-assessment/1.0`
- `selectionMode`: `FIXED_POC_DEFAULT | CALLER_PROVIDED`
- `itemCount=3`, `itemTimeLimitSeconds=180`, `replaySpeed=60`
- ordinal이 `1, 2, 3`인 세 문항
- 각 문항의 ScenarioVersion checksum, 단일 가명 자산, 짧은 명세, 캔들 240개와 뉴스 전체

응답은 사용자나 세션을 만들지 않으며 동일 요청의 결과를 저장하지 않는다.

### `POST /v1/poc/snapshot`

프론트는 시험이 끝나면 다음 자료를 한 번에 보낸다.

- `questionnaireVersionId`와 10개 원본 `answers`
- ordinal `1, 2, 3`의 서로 다른 세 scenario와 assessment에서 받은 version·checksum
- item별 `completionReason`, `finalElapsedMs`, 시간순 `events[]`

엔진은 client가 보낸 가격·캔들·뉴스를 신뢰하지 않는다. `scenarioId`로 서버의 검수 catalog를
다시 찾고 version·checksum을 대조한 뒤 판단 당시 가격, 뉴스와 평가 문맥을 조립한다. 응답은
`snapshotHash`와 `payload`만 포함하며 사용자 ID, 세션 ID, Snapshot ID와 절대 server
timestamp를 포함하지 않는다.

---

## 성향 설문 계약

평가 시작 전 다음 10개 항목에 응답한다.

- 투자 경험
- 위험 선호도
- 레버리지 상품 특성 이해도
- 불리한 가격 움직임에 대한 대응 성향
- 정보 출처 검증 습관
- 관심 자산 및 산업
- 평가 중 감수 가능한 불확실성 수준
- 주로 고려하는 투자 기간
- 가격 방향 판단 시 우선 확인하는 근거
- 평가에서 확인하고 싶은 학습 목표

현재 게시 설문은 `questionnaire-safe-t-v2`다.

- 모든 문항은 필수다.
- 단일 선택 문항은 정확히 하나를 선택한다.
- 관심 분야는 1개 이상 3개 이하를 선택한다.
- 학습 목표는 1개 이상 2개 이하를 선택한다.
- 엔진은 질문·선택지와 선택 범위만 검증한다.
- 초기 설문 원본은 최종 Snapshot에 질문 문구와 사람이 읽을 수 있는 답변 label로 함께 넣는다.

성향 요약, 강점·취약점 분석과 문항 추천을 LLM으로 수행한다면 이는 외부 Selection Service의
책임이다. Backend Engine에는 prompt, model, AI SDK 또는 성향별 위험 한도가 없다.

---

## 시험과 Runtime Scenario

한 번의 시험은 순서가 고정된 독립 시장 판단 문항 정확히 세 개로 구성한다.

- 각 문항은 단일 가명 자산 하나를 다룬다.
- 문항 사이에 의견이나 자산 상태를 이월하지 않는다.
- 프론트는 앞 문항을 닫은 뒤 다음 문항을 표시한다.
- 각 문항에는 UP 또는 DOWN 판단을 최소 한 번 남기는 것이 요구된다.
- 제한 시간 동안 같은 방향 재확인과 다른 방향 후속 판단을 횟수 제한 없이 남길 수 있다.
- 이미 기록된 판단을 수정한다는 개념은 없고 각 입력은 새 이벤트다.

각 Runtime Scenario는 다음으로 구성한다.

1. 단일 가명 자산과 한 단락 이내의 짧은 사실 명세
2. TradingView Lightweight Charts 형식의 1분 OHLCV 캔들 240개
3. 시험 중 공개되는 가명 뉴스·공시 한 건 이상
4. `timeLimitSeconds=180`, `replaySpeed=60`

각 뉴스는 제목·본문·공개 offset과 함께 `sourceLabel`, 구조화된 `sourceType`,
`informationRole`, `isSimulationContent`를 제공한다. 프론트는 이를 이용해 공시·정부 발표·보도·
공식 게시물을 구분하고 시뮬레이션 콘텐츠 라벨을 표시한다. 후단 평가는 1차 사건·맥락·정정·
동시대 보도의 역할을 문자열에서 추측하지 않고 확인할 수 있다.

캔들 240개는 시작 화면의 pre-roll 60개와 시험 시장 구간 180개다. `marketOffsetMs`는 pre-roll
`-3,600,000..-60,000`, 시험 구간 `0..10,740,000`을 나타낸다. 프론트는 캔들의
`availableAtOffsetMs`와 뉴스의 `availableAtOffsetMs`가 현재 시장 offset 이하일 때 화면에
표시한다.

### 시나리오 제작과 가명화

각 문항은 특정 실제 자산의 과거 사건 자료를 기반으로 한다.

```text
실제 OHLCV + 사건 명세 + 당시 뉴스
→ 자산·기업·인물·날짜 가명화
→ 가격의 결정론적 정규화
→ 외부 LLM으로 뉴스 제목·본문 재작성
→ 사람의 사실·미래정보·재식별 위험 검수
→ 버전이 고정된 Runtime Scenario
```

가격은 source 시작 가격 `P0`와 시나리오에 고정된 비식별 시작값 `B`로 정규화한다.

```text
정규화 가격(t) = 실제 가격(t) / P0 × B
```

모든 OHLC에 같은 배율과 버전 고정 반올림 규칙을 적용하고 상대 변화율과 OHLC 대소 관계를
보존한다. LLM은 캔들·가격·거래량을 생성하거나 임의 수정하지 않는다.

뉴스 재작성 LLM은 실제 당시 자료의 확인된 사실과 사건 순서를 보존하면서 실존 식별자와 실제
날짜를 제거한다. 원문 URL, 원문 hash와 alias map은 Runtime Scenario, API와 Evaluation
Snapshot에 넣지 않는다. 사람의 승인을 거친 시나리오만 PoC catalog에 포함한다.

Scenario Builder LLM은 오프라인 제작 도구이며 Runtime Backend Engine의 일부가 아니다.

---

## 3분·60배속과 프론트 상태

각 문항의 표시 제한 시간은 180초이고 시장은 서비스 시간보다 60배 빠르게 진행한다.

```text
서비스 1분 = 시장 시간 60분
문항 3분 = 시장 시간 180분 = 10,800,000ms
marketOffsetMs = elapsedMs × 60
```

프론트는 다음 상태를 메모리에서 관리한다.

- 현재 ordinal과 문항별 `elapsedMs`
- 180초 timer와 60배속 차트 공개 범위
- 판단과 뉴스 열람의 전역 `sequence`
- item별 이벤트 목록, 완료 이유와 최종 경과 시간
- assessment에서 받은 각 scenario의 version·checksum
- 문항 완료/다음과 전체 시험 종료 버튼

`elapsedMs`는 `0..180000` 범위다. `TIMEOUT`인 문항은 `finalElapsedMs=180000`이어야 한다.
판단이 한 건 이상이고 `finalElapsedMs < 180000`인 문항만 `USER_COMPLETED`로 조기 완료할 수
있다. `ASSESSMENT_FINISHED`도 `finalElapsedMs < 180000`에서만 유효하다. 정확히 180초에
닫힌 문항은 `TIMEOUT`이다. 전체 시험 종료 시 현재와
남은 문항은 `ASSESSMENT_FINISHED`로 제출할 수 있다. 첫 `ASSESSMENT_FINISHED` 뒤의 문항은
`finalElapsedMs=0`과 빈 events를 가져야 하며 시험 종료 뒤 행동을 다시 추가할 수 없다.

서버는 최종 bundle에서 값의 범위, 사건 순서와 시나리오 공개 offset을 검증하지만 실제 버튼을
그 시각에 눌렀는지 증명하지 않는다. 새로고침하면 진행 상태가 사라질 수 있고 재접속 복원도
없다. 세 문항의 미래 캔들·뉴스가 최초 assessment 응답에 모두 포함되므로 개발자 도구를 통한
선조회도 막지 않는다. 이는 가명화된 데모를 위한 한계이며 운영형 WIP에서 보완한다.

---

## 판단과 뉴스 열람 이벤트

최종 제출의 각 이벤트에는 `sequence`, `elapsedMs`, `type`이 필요하다.
sequence와 시간 값은 coercion하지 않는 JSON integer다.
행동이 한 건도 없는 item도 `events` 필드를 생략하지 않고 빈 배열로 제출한다.

### 판단 이벤트

```json
{
  "sequence": 1,
  "elapsedMs": 10000,
  "type": "JUDGMENT",
  "direction": "UP",
  "confidence": "MEDIUM",
  "reasonTags": ["PRICE", "VOLUME"],
  "reasonText": "거래량을 동반한 반등으로 판단했다."
}
```

- `direction`: `UP | DOWN`
- `confidence`: `LOW | MEDIUM | HIGH`, 필수
- `reasonTags`: `PRICE | VOLUME | SCENARIO_BRIEF | NEWS | INTUITION | OTHER` 중 하나 이상, 필수
- `reasonText`: 선택, trim·Unicode NFC 정규화 후 최대 500자

`UP`은 현재 공개된 정보를 보고 앞으로 상승할 것 같다는 의견이고 `DOWN`은 하락할 것 같다는
의견이다. 매수·매도나 long·short가 아니며, 사후 가격의 정답 라벨도 아니다.

한 문항에서 다음처럼 여러 번 판단할 수 있다.

```text
UP → UP → DOWN
```

이는 앞의 UP을 수정한 것이 아니라 서로 다른 세 판단 증거다. 단순히 여러 번 눌렀거나 방향을
바꿨다는 사실 자체를 정답·오답으로 취급하지 않는다.

### 뉴스 열람 이벤트

```json
{
  "sequence": 2,
  "elapsedMs": 20000,
  "type": "CONTENT_VIEW",
  "contentId": "news-a"
}
```

뉴스 목록이 화면에 나타난 것과 사용자가 상세 제목·본문을 실제로 연 것을 구분한다.
`CONTENT_VIEW`는 해당 뉴스의 `availableAtOffsetMs` 이후에만 유효하다. 같은 뉴스를 다시 연
사실도 별도 이벤트로 제출할 수 있다.

모든 문항의 판단과 열람 이벤트는 하나의 전역 `sequence`를 공유한다. 값은 1부터 시작해 중복과
누락 없이 연속하고, 각 item 안에서는 `sequence`와 `elapsedMs`가 오름차순이어야 한다. 뒤 문항의
이벤트가 시작된 후 앞 문항으로 돌아갈 수 없다.

예를 들어 다음 원본 행동 순서를 그대로 표현할 수 있다.

```text
sequence 1: UP
sequence 2: 뉴스 A 열람
sequence 3: UP
sequence 4: 뉴스 B 열람
sequence 5: DOWN
```

Snapshot은 판단을 `responses[]`, 열람을 `contentViews[]`로 분리하되 두 배열에 같은 sequence를
보존한다. 외부 평가기는 두 배열을 합쳐 sequence 오름차순으로 정렬하는 결정론적 전처리만으로
원래 행동 순서를 복원한다. 한 원본 행동은 두 배열 중 정확히 한 곳에 한 번만 들어간다.

### 답변 상태

- 판단이 한 건 이상이면 `ANSWERED`이며 LLM 채점이 가능하다.
- 판단이 한 건도 없으면 `UNANSWERED`이며 후단 규칙이 문항 점수 0을 적용한다.
- 뉴스 열람만으로는 `ANSWERED`가 되지 않는다.
- 조기 완료 자체에 고정 감점은 없다.
- 선택적 `reasonText`를 생략했다는 사실만으로 감점하지 않는다.

---

## Evaluation Snapshot v5

`POST /v1/poc/snapshot`은 최종 client bundle을
`safe-t-evaluation-snapshot/5.0`으로 변환한다. Backend Engine은 Snapshot을 저장하거나 다시
조회하지 않는다.

응답 envelope는 다음처럼 내용 hash와 payload만 갖는다.

```json
{
  "snapshotHash": "sha256:...",
  "payload": {
    "schemaVersion": "safe-t-evaluation-snapshot/5.0"
  }
}
```

Snapshot에는 다음 평가 근거를 포함한다.

- 설문 버전·checksum, 질문 문구, 원본 답과 사람이 읽을 수 있는 label
- 정확히 세 item의 ordinal, ScenarioVersion과 checksum
- item별 `completionReason`, 최종 시장 offset, `ANSWERED/UNANSWERED`
- item별 실제 최종 관찰시간
- 가명 자산, 짧은 시나리오 명세와 가명화 상태
- item별 모든 반복 판단과 필수 태그·확신도·선택 이유
- item별 모든 상세 뉴스 열람 이벤트
- item 종료까지 공개된 모든 뉴스의 제목·본문·출처 유형·정보 역할·시뮬레이션 여부와
  열람 여부·횟수·최초 열람 sequence를 한 번씩 담은 `newsTimeline[]`
- 두 이벤트 배열이 공유하는 전역 sequence
- 판단 당시 가격과 기준 캔들
- 각 판단 기준 직전 10개·기준 1개·직후 10개의 1분봉 window
- 판단 직전·직후의 변화율, 고저 범위와 평균 거래량
- 판단 시점까지 이용 가능했던 뉴스와 그전 실제 열람 여부·최초 열람 sequence
- 판단 뒤 window 중 item 종료 전에 새로 도착해 당시에는 알 수 없었던 뉴스
- 엔진·canonicalization·평가 문맥 정책 버전

프론트 재생용 캔들 240개와 종료 뒤 미래 뉴스는 Snapshot에 복사하지 않는다. 뉴스 제목·본문은
item-level timeline에 한 번만 두고 판단 문맥과 열람 이벤트는 `contentId`로 참조한다. 각 판단에
필요한 작은 시장 문맥만 넣어 외부 LLM이 별도 DB 조회나 가격 계산 없이 평가할 수 있게 한다.
판단 후 캔들은 판단 과정의 효용을 해석하는 문맥이며 정답·오답 근거가 아니다.

첫 `ASSESSMENT_FINISHED` item은 종료 버튼을 누른 현재 문항이므로 그 시점의 문맥을 보존한다.
그 뒤 item은 미개봉이므로 시작 시점 공개 뉴스가 정적 package에 있어도 `newsTimeline=[]`다.
판단 뒤 새 뉴스 범위는 직후 마지막 candle의 완성 시각까지로 계산하되 item 종료 시점을 넘지
않는다.

Snapshot에는 다음을 넣지 않는다.

- 사용자, 참가자, 설문 제출, 세션, item event 또는 Snapshot의 인스턴스 ID
- `startedAt`, `deadlineAt`, `recordedAt`, `endedAt`, `generatedAt` 같은 절대 server timestamp
- LLM prompt·model, 점수, PASS와 보고서
- 포지션·포트폴리오·수익률·정답 방향
- 실제 원천 자산·날짜·URL·alias map

payload를 RFC 8785/JCS로 canonicalize한 뒤 SHA-256 `snapshotHash`를 계산한다. 저장 불변성이
아니라 **내용 주소화된 값 불변성**이다. 같은 정규화 입력과 같은 정적 scenario/questionnaire
version은 같은 payload와 hash를 만든다.

---

## 외부 LLM 과정 평가

Backend Engine은 LLM을 호출하지 않고 점수나 PASS를 만들지 않는다. 외부 Evaluation Service는
Snapshot v5와 공개된 rubric을 입력으로 사용한다.

### 문항 완료 gate

- `UNANSWERED`: 해당 item의 최종 점수는 정확히 `0`
- `ANSWERED`: 고정 rubric에 따라 LLM이 항목별 점수와 근거를 구조화해 반환
- `USER_COMPLETED`: 이 완료 이유 자체로 감점하지 않음

완료 여부와 0점 적용은 LLM 추론이 아니라 후단 규칙 코드의 책임이다.

### 항목별 평가

답변한 각 item은 100점 만점으로 다음 항목을 평가한다.

| 평가 항목 | 배점 | 판단 증거 |
|---|---:|---|
| 위험 신호 인식과 대응 | 20 | 변동성·거래량·뉴스의 위험 신호를 응답 근거에 반영했는지 |
| 불확실성과 변동성 인식 | 15 | 당시 정보의 불확실성과 가격 변동성을 판단 태그·확신도에 반영했는지 |
| 정보 활용과 출처 구분 | 15 | 시나리오 설명과 가명 뉴스의 역할을 구분하고 필요한 상세 정보를 열람했는지 |
| 시장 흐름 해석 | 15 | 당시 공개된 캔들·거래량·사건을 UP/DOWN 판단과 연결했는지 |
| 초기 성향과 응답의 정합성 | 15 | 설문에서 밝힌 정보 활용·위험 인식 성향과 실제 응답이 어떻게 이어지는지 |
| 판단·확신도·근거의 일관성 | 10 | 판단·확신도·근거와 새 정보 이후의 변화가 일관되는지 |
| 판단 근거의 유용성 | 10 | 선택한 근거가 당시 의사결정에 실질적으로 유용하고 서로 모순되지 않는지 |
| **합계** | **100** |  |

판단의 효용은 가격을 맞혔는지가 아니다. 당시 이용 가능했던 정보에서 근거를 선택하고 확신도를
표현했는지, 시간에 따른 변화가 새 정보와 시장 맥락에 연결되는지를 평가한다.

### 총점과 PASS

- 문항별 점수 범위: `0..100`
- 최종 점수: `(item1Score + item2Score + item3Score) / 3`
- 반올림 전 최종 점수 `70` 이상이면 PASS
- 각 항목 점수는 `0..배점` 범위의 소수점 한 자리이며 문항 점수는 항목 합계
- 범위·합계 검증, 미응답 0점, 평균과 PASS는 규칙 코드가 수행
- LLM 결과가 누락되거나 범위를 벗어나면 clamp하지 않고 실패로 처리

미응답 item 하나가 있으면 이론상 최고 최종 점수가 `66.666...`이므로 PASS할 수 없다. 방향
적중 여부와 사후 가격 수익률은 계산·표시·채점하지 않는다.

평가 서비스도 이번 stateless PoC에서는 결과를 사용자별로 저장하지 않는다. 필요하면
`snapshotHash`, rubric·prompt·model version을 호출자가 결과와 함께 보관할 수 있다.

---

## 시스템 책임 경계

| 주체 | 처리 내용 |
|---|---|
| Frontend | 10문항 답과 선택된 세 scenario를 메모리에 유지하고 3분 timer·60배속·순차 진행·행동 sequence를 관리 |
| 외부 Selection Service | 선택 사용 시 설문과 `scenario-catalog`을 분석하고 정확히 세 scenario ID를 제공 |
| 외부 Scenario Builder | 실제 자료 수집, 가격 정규화, 식별자 가명화와 LLM 뉴스 재작성 |
| Stateless Backend Engine | 질문지·선택 후보 반환, 설문 검증, scenario 전달, 최종 bundle 검증, 판단별 평가 문맥과 Snapshot v5/hash 생성 |
| Evaluation LLM | 답변 item의 항목별 점수·근거와 개선 의견 생성 |
| Result Rules | 미응답 0점, 점수 범위·합계 검증, 평균과 PASS 결정 |
| Frontend Result View | 점수·분석·PASS를 현재 화면에 표시 |

Backend Engine에는 사용자 profile, 인증, DB, server session, timer scheduler, LLM API key, prompt,
model 설정과 평가 결과 저장소가 없다.

---

## PoC 제외와 운영형 WIP

다음은 현재 stateless PoC에서 제외한다.

- 참가자 계정·사용자 ID·인증·인가
- 설문 제출, 세션, 행동, Snapshot과 평가 결과의 DB 저장
- 세션 재접속·새로고침 복원·여러 기기 동기화
- 권위 있는 server timer와 건별 행동 수락 시각
- 미래 item 잠금과 server-side streaming/delta 공개
- event별 멱등 저장과 수정·삭제 방지 감사 로그
- 영속 PASS 인증 코드, 발급 이력과 공개 검증 페이지
- 실제 돈 입출금과 증권사 계좌 연동
- 가상 자금, 주문, 체결, 보유량, 비중, 현금과 포트폴리오 상태
- PnL·수익률·MDD·레버리지·노출·집중도
- 실시간 종목 추천과 사용자 간 수익률 경쟁

운영형 버전에서는 사용자 동의와 최소 수집 원칙 아래 인증·영속 상태, 서버 시각, 재접속 복원,
미래 데이터 차단, 요청 멱등성과 감사 가능성을 별도 명세·schema·migration으로 추가한다. 이를
추가하기 전에는 현재 `UP/DOWN`에서 포지션이나 거래를 암묵적으로 추론하지 않는다.

---

## PoC 필수 비기능 조건

- 질문지, Runtime Scenario, Snapshot schema와 평가 rubric은 명시적으로 versioning한다.
- Backend Engine은 시작 시 질문 ID·순서·선택 범위와 Runtime Scenario 파일의 선언 SHA-256,
  candle phase·시간축, 뉴스 공개 시각·offset 정합성을 검증한다.
- API 입력은 정의되지 않은 필드를 거부하고 설문, 세 scenario, 시간 범위와 전역 sequence를
  엄격하게 검증한다.
- client가 보낸 가격·뉴스 본문 대신 검수된 서버 정적 자료로 Snapshot 평가 문맥을 만든다.
- assessment에서 전달한 scenario version·checksum을 최종 제출에서 다시 받아 현재 package와
  일치하는지 검사한다.
- Snapshot payload는 canonicalize하고 hash를 검증할 수 있어야 한다.
- API와 Snapshot에는 사용자·세션 식별자와 절대 server timestamp를 넣지 않는다.
- 외부 LLM 가명화 결과는 사람이 승인하기 전 Runtime Scenario로 사용하지 않는다.
- 원천 자료와 alias map은 Runtime Backend Engine에서 분리한다.
- 모든 화면과 결과에 투자 추천이 아닌 교육용 시뮬레이션임을 표시한다.
- PoC에서는 실명, 실제 계좌와 실제 자산 규모를 수집하지 않는다.
- 문서와 UI는 timer·순차 진행·행동 원본성이 프론트 신뢰에 의존한다는 한계를 숨기지 않는다.
