# 구현 저장소 정합성 기록

> 갱신 2026-09-05 · 기준: `spec/README.md`, 단일 Backend `0.7.0`

## 결론

현재 배포 저장소는 설문부터 PASS 결과까지 한 프로세스에서 제공하는 stateless end-to-end PoC
Backend다.

```text
고정 10문항 설문
→ OpenAI Selection structured output: 성향 분석 + scenario 3개
→ 중앙 엔진: 설문 검증 + replay package
→ 브라우저: 180초 × 3문항 순차 재생 + 최종 events 기록
→ 중앙 엔진: 기록 검증 + content-addressed Snapshot v5
→ OpenAI Evaluation structured output: 답변 item별 7항목
→ Result Rules: 미응답 0점 + 3문항 평균 + 70점 PASS
→ stateless INVEST PASS façade
```

배포 단위는 하나지만 책임 경계는 합치지 않는다. Selection과 Evaluation LLM은 구조화된 해석만
만들고, 중앙 엔진은 설문·scenario version·replay offset·이벤트 순서와 Snapshot 사실을
결정론적으로 검증한다. Result Rules는 LLM 점수의 범위와 합계를 검증한 뒤 평균과 PASS를
결정한다. 어느 LLM도 중앙 엔진의 Snapshot을 수정하거나 규칙을 우회할 수 없다.

사용자 ID, 인증, DB, 서버 세션, 단건 이벤트 저장, Snapshot·평가 결과·PASS 저장은 없다.
호출자는 진행 상태를 브라우저 메모리에 유지하고 최종 요청에 필요한 전체 묶음을 다시 보낸다.

## 공개 API

### Canonical frontend 3-call

| 순서 | Method | Path | 역할 |
|---:|---|---|---|
| 1 | `GET` | `/v1/poc/questionnaire` | `questionnaire-safe-t-v2` 10문항 반환 |
| 2 | `POST` | `/v1/poc/onboarding-assessment` | 설문 분석, 서로 다른 scenario 3개 선택과 전체 replay package 반환 |
| 3 | `POST` | `/v1/poc/evaluation` | 최종 bundle 검증, 내부 Snapshot 생성, 7항목 평가와 PASS 반환 |

두 번째 응답은 profile analysis, 선택 이유와 catalog·prompt·model provenance, 세 scenario의 전체
캔들·뉴스를 함께 담는다. 세 번째 요청은 원본 설문과 ordinal `1..3` item의 version·checksum,
completion과 최종 events를 다시 담는다. 평가 응답은 `snapshotHash`, item별 분석, 최종 점수,
PASS/FAIL과 조건부 `INVEST PASS` artifact를 포함하지만 서버에 남지 않는다.

### Low-level primitives

| Method | Path | 역할 |
|---|---|---|
| `GET` | `/v1/poc/scenario-catalog` | Selection 입력용 후보 ID·type·brief·version·checksum 반환 |
| `POST` | `/v1/poc/assessment` | 지정한 서로 다른 scenario 3개 또는 명시적 고정 기본 세트 반환 |
| `POST` | `/v1/poc/snapshot` | 평가 없이 최종 bundle을 Snapshot v5/hash로만 변환 |

저수준 primitive는 조합, 계약 검사와 디버그용이다. `/assessment`에서 `scenarioIds`를 생략해
`FIXED_POC_DEFAULT`를 요청하는 것은 명시적 기능이며 Selection 실패 fallback이 아니다.

`/`, `/health`, `/ready`는 프로세스와 정적 자료·LLM 설정 준비 상태를 확인한다. 모든 endpoint는
Authorization, 참가자 header, cookie와 `Idempotency-Key` 없이 동작하며 사용자별 resource를
생성하지 않는다.

## OpenAI runtime 계약

- `OPENAI_API_KEY`, `OPENAI_MODEL`, `OPENAI_REASONING_EFFORT`는 프로세스 시작에 필수다.
  `OPENAI_TIMEOUT_SECONDS`는 선택 설정이다.
- 앞단 Selection은 실제 OpenAI Responses API의 Pydantic structured output을 사용한다.
- 후단 Evaluation은 실제 OpenAI Responses API의 strict JSON Schema structured output을
  사용한다.
- 두 호출 모두 `store=false`이며 `conversation`이나 `previous_response_id`를 사용하지 않는다.
- Selection과 Evaluation은 각각 애플리케이션 수준 최대 3회만 시도한다.
- 모델 요청이나 출력 검증이 끝내 실패하면 `502`를 반환한다. 고정 scenario, 임의 점수, PASS나
  다른 모델로 자동 대체하지 않는다.
- API key와 원시 SDK 오류 문자열은 public 오류 응답, Snapshot과 결과 provenance에 넣지 않는다.
  응답에는 비밀값이 아닌 model·reasoning effort·prompt version만 표시할 수 있다.

## 일치하는 시험·기록 계약

| 영역 | 0.7 구현 계약 |
|---|---|
| 시험 | 단일 가명 자산 문항 정확히 3개, 프론트 순차 진행 |
| 시간 | 문항당 180초, 시장 60배속, 프론트 메모리 timer |
| 응답 | 문항당 `UP/DOWN` 판단을 여러 번 append; 한 건 이상이면 `ANSWERED` |
| 응답 근거 | 각 판단에 `reasonTags` 1개 이상과 `confidence` 필수, `reasonText` 선택 |
| 설문 | 10문항 원본 답을 onboarding과 최종 evaluation에 각각 제출 |
| 시나리오 선택 | version·checksum 고정 catalog 안에서 정확히 서로 다른 ID 3개를 순서대로 선택 |
| 열람 | 뉴스 목록 노출과 실제 상세 열람 `CONTENT_VIEW`를 구분 |
| 순서 | 판단과 열람이 시험 전역의 누락·중복 없는 연속 `sequence`를 공유 |
| 완료 | `USER_COMPLETED`, `TIMEOUT`, `ASSESSMENT_FINISHED` 상태와 `finalElapsedMs`를 검증 |
| 제외 | 포지션·주문·잔고·비중·PnL·MDD·별도 지식 quiz 없음 |

`UP/DOWN`은 각 시점의 방향 의견이지 매수·매도, long·short 또는 사후 정답 label이 아니다.
반복 판단이나 방향 변경 자체도 정답·오답으로 보지 않는다.

## 중앙 엔진과 Snapshot v5 무결성 경계

- assessment가 준 `scenarioVersionId`와 `scenarioChecksum`을 최종 요청에서 echo하고 현재 정적
  package와 대조한다.
- client가 가격·캔들·뉴스를 제출하지 못하게 하고 `scenarioId`로 서버 정적 자료를 다시 찾는다.
- event `elapsedMs`, 전역 `sequence`, item 순서, content 공개 offset과 완료 상태를 검증한다.
- `responses[]`와 `contentViews[]`는 같은 sequence를 보존하므로 합쳐 원본 행동 순서를 복원할
  수 있다.
- `newsTimeline[]`에는 item 종료까지 공개된 읽은 뉴스와 읽지 않은 뉴스를 각각 한 번 넣고,
  출처 유형·정보 역할·열람 횟수·최초 열람 sequence를 제공한다.
- 판단별 기준 candle과 직전 최대 10개·직후 최대 10개, 당시 이용 가능 뉴스와 선행 열람 여부를
  계산한다. 종료 뒤 미래 정보는 Snapshot에 넣지 않는다.
- 미개봉 item과 미응답 item도 제거하지 않고 상태와 빈 배열을 명시한다.
- Selection profile은 저장하거나 Snapshot에 복사하지 않는다. Evaluation은 Snapshot의 원본 설문
  응답과 실제 행동만 직접 대조한다.
- payload를 RFC 8785/JCS로 canonicalize하고 SHA-256 `snapshotHash`를 계산한다. 이는 DB 저장
  불변성이 아니라 동일 입력·정적 release에 대한 내용 불변성이다.

중앙 엔진은 브라우저가 실제로 해당 시간에 화면을 봤는지, event가 발생 즉시 기록됐는지는
증명하지 않는다. 이 한계를 숨기지 않되 LLM이 client 주장이나 사후 방향 적중을 새로운 평가
사실로 만드는 것은 허용하지 않는다.

## 7항목 평가와 PASS

답변한 각 item은 다음 고정 rubric으로 100점 만점 평가를 받는다.

| criterion | 배점 |
|---|---:|
| 위험 신호 인식과 대응 | 20 |
| 불확실성과 변동성 인식 | 15 |
| 정보 활용과 출처 구분 | 15 |
| 시장 흐름 해석 | 15 |
| 초기 성향과 응답의 정합성 | 15 |
| 판단·확신도·근거의 일관성 | 10 |
| 판단 근거의 유용성 | 10 |
| **합계** | **100** |

- 판단이 없는 `UNANSWERED` item은 LLM에 보내지 않고 Result Rules가 정확히 0점을 적용한다.
- 답변 item의 criterion 점수는 각 배점 범위 안이어야 하고 item 합계와 정확히 일치해야 한다.
- 최종 점수는 `(item1 + item2 + item3) / 3`이며 반올림 전 평균이 `70` 이상일 때 PASS다.
- 한 item이 미응답이면 이론상 최고 평균이 `66.666...`이므로 PASS할 수 없다.
- 방향 적중률, 사후 수익률과 정답 방향은 계산·표시·채점하지 않는다.
- PASS artifact는 `snapshotHash`, rubric·prompt·model·result-rule version에 묶인 현재 응답일
  뿐이며 영속 자격증이나 실제 투자 적격성이 아니다.

## 현재 자료와 의도적인 PoC 한계

Runtime은 `data/historical-scenarios/v2`를 읽기만 한다. 이 raw 자료는 자산·시각·뉴스가
가명화됐지만 가격이 아직 원본 범위여서 모든 candidate가 `participantSafe=false`다. 로컬
통합·검수용이며 가격 정규화와 사람 검수가 끝나기 전 외부 참가자에게 공개하면 안 된다.

세 scenario의 240개 캔들과 뉴스 전체를 assessment 응답 하나로 보내므로 개발자 도구를 통한
미래 데이터 선조회를 막지 않는다. 서버는 권위 있는 timer, event append 시각, 새로고침·재접속
복구와 여러 기기 동기화를 제공하지 않는다.

사용자별 mutable state가 없어 정상 동시 요청의 데이터가 섞이지 않고 동일 정적 release를 읽는
여러 worker는 같은 Snapshot hash를 만든다. 다만 공개 배포의 rate/body/concurrency 제한, TLS,
OpenAI quota와 모니터링은 인프라 책임이다.

다음은 계속 범위 밖이다.

- 참가자 계정·사용자 ID·인증·인가
- 설문, 세션, 행동, Snapshot, 평가 결과와 PASS의 DB·파일 저장
- server timer, event별 ACK·polling·streaming과 감사 로그
- 영속 PASS 코드, 발급 이력과 공개 검증 페이지
- 실제 돈·증권 계좌, 가상 자금·주문·체결·포지션·포트폴리오
- 수익률·MDD·레버리지·노출·집중도와 실시간 종목 추천

## Frontend 전달 사항

- canonical frontend는 questionnaire → onboarding-assessment → evaluation의 3-call만 사용한다.
- onboarding 응답의 세 scenario version·checksum, 현재 ordinal, item timer와 전역 events를
  브라우저 메모리에 유지한다.
- evaluation 요청에는 같은 원본 설문과 정확히 세 item의 최종 completion·events를 보낸다.
- 결과 화면은 7항목 분석, 평균, PASS/FAIL과 조건부 PASS artifact를 현재 응답에서 바로 표시한다.
- 저수준 catalog·assessment·snapshot primitive를 정상 사용자 흐름의 추가 polling 단계로 만들지
  않는다.
- 운영형 인증·저장·server timer·미래 데이터 차단·멱등성과 감사 기능은 별도 API와 schema로
  설계한다.
