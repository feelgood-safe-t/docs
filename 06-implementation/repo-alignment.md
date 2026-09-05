# 구현 저장소 정합성 기록

> 갱신 2026-09-05 · 기준: `spec/README.md`, Backend Engine `0.6.0`

## 결론

Backend Engine은 사용자별 애플리케이션 서버가 아니라 다음 네 기능만 수행하는 stateless
비-LLM PoC API다.

```text
고정 설문·검수된 선택 후보 catalog 제공
→ 설문 검증 후 시나리오 3개의 캔들·뉴스 일괄 전달
→ 프론트가 모은 최종 행동 묶음을 Evaluation Snapshot v5로 변환
```

사용자 ID, 인증, DB, 서버 세션, 단건 이벤트 저장, LLM 호출, 채점, PASS 발급은 포함하지
않는다. 기존 Selection Orchestrator·성향 분류 모델·LLM Scenario Builder 구현도 backend
저장소에서 제거했으며, 필요하면 별도 서비스나 오프라인 도구로 구성한다.

## 공개 API

| Method | Path | 역할 |
|---|---|---|
| `GET` | `/v1/poc/questionnaire` | `questionnaire-safe-t-v2` 10문항 반환 |
| `GET` | `/v1/poc/scenario-catalog` | Selection LLM용 후보 ID·type·brief·version 반환 |
| `POST` | `/v1/poc/assessment` | 설문 검증 후 서로 다른 시나리오 3개 전체 전달 |
| `POST` | `/v1/poc/snapshot` | 설문과 최종 events를 Snapshot v5/hash로 변환 |

`/`, `/health`, `/ready`는 프로세스 상태 확인용이다. Authorization, 참가자 header,
cookie와 Idempotency-Key는 사용하지 않는다.

## 일치하는 계약

| 영역 | 제품 계약과 구현 |
|---|---|
| 시험 | 단일 자산 문항 정확히 3개, 프론트 순차 진행 |
| 시간 | 문항당 180초, 시장 60배속, 프론트 메모리 timer |
| 응답 | 문항당 UP/DOWN을 수시로 append; 1건 이상이면 `ANSWERED` |
| 응답 근거 | `reasonTags` 1개 이상과 `confidence` 필수, `reasonText` 선택 |
| 설문 | 10문항 원본 답을 assessment와 snapshot에 각각 제출 |
| 시나리오 선택 | 버전·checksum 고정 후보 catalog에서 외부 선택기가 3개 ID 생성 |
| 열람 | 뉴스 목록 노출과 실제 상세 열람 `CONTENT_VIEW`를 구분 |
| 순서 | 판단과 열람이 시험 전역의 연속 `sequence`를 공유 |
| Snapshot | 설문, 세 시나리오, 모든 판단·열람, 판단별 캔들 문맥과 뉴스 상태 |
| 평가 경계 | 방향 적중·수익률을 계산하지 않으며 LLM·점수·PASS는 후단 책임 |
| 제외 범위 | 사용자·인증·상태·포지션·비중·잔고·PnL·MDD·별도 지식 문항 없음 |

## Snapshot v5 보완

- assessment가 준 `scenarioVersionId`와 `scenarioChecksum`을 최종 요청에서 echo하고 현재
  정적 package와 대조한다.
- `responses[]`와 `contentViews[]`는 같은 sequence를 보존하므로 합쳐 정렬할 수 있다.
- `newsTimeline[]`에는 item 종료까지 공개된 뉴스를 읽지 않은 항목까지 한 번씩 넣고,
  출처 유형·정보 역할·열람 여부·횟수·최초 sequence를 제공한다.
- 뉴스 제목·본문은 timeline에 한 번만 두고 다른 배열은 `contentId`로 참조한다.
- 판단별 캔들은 직전 최대 10개, 기준 1개, 직후 최대 10개만 포함한다.
- `finalRealElapsedMs`와 `finalMarketOffsetMs`를 함께 제공한다.
- 판단 0건인 item도 `UNANSWERED`로 남겨 후단이 정확히 0점을 적용할 수 있게 한다.
- 첫 `ASSESSMENT_FINISHED` 뒤의 미개봉 item은 시작 뉴스가 있어도 `newsTimeline=[]`다.
- `USER_COMPLETED`와 `ASSESSMENT_FINISHED`는 180초 미만, `TIMEOUT`은 정확히 180초이며
  빈 행동도 `events=[]`로 명시한다.

## 의도적인 PoC 한계

세 시나리오의 240개 캔들과 뉴스 전체를 assessment 응답 하나로 보내므로 개발자 도구를 통한
미래 데이터 선조회를 막지 않는다. 서버는 클라이언트가 주장한 event 시각과 append-only
원본성을 증명하지 않고, 새로고침·재접속도 복구하지 않는다.

기본 v2 자료는 자산명·시각·뉴스를 가명화했지만 가격은 아직 원본 범위라
`participantSafe=false`다. 외부 공개 전에 가격 정규화와 사람 검수가 필요하다.

사용자별 mutable state가 없으므로 동시 요청이 섞이지 않고, 동일 정적 release를 읽는
여러 worker도 같은 Snapshot hash를 낸다. Compose worker는 `SAFE_T_WORKERS`로 조정한다.
공개 배포의 rate/body/concurrency 제한, TLS와 모니터링은 인프라 책임이다.

## 저장소 간 전달 사항

- 프론트는 `GET /v1/poc/questionnaire`를 사용하고 문항 timer·현재 ordinal·events를 메모리에
  보관한다.
- 시험 종료 시 설문과 정확히 세 item을 `POST /v1/poc/snapshot`에 한 번 보낸다.
- 개인화 선택이 필요하면 외부 서비스가 `scenario-catalog`을 보고 검수된 scenario ID
  세 개만 assessment 요청에 넣는다.
- Evaluation Service는 Snapshot v5를 받아 항목별 점수를 만들며, 미응답 0점·평균·PASS는
  결정론적 후단 규칙이 처리한다.
- 운영형 사용자 식별, 인증, 저장, server timer, streaming 공개와 감사 로그는 별도 API와
  schema로 설계한다.

## Frontend 연동 현황

2026-09-05 기준 `feelgood-safe-t/frontend@271840a`의 API 모드는 아직 이전 상태형 backend 계약을
사용하므로 현재 PoC Backend Engine과 직접 연동되지 않는다.

- `/v1/participants/guest`, `/v1/survey-submissions`, `/v1/assessment-sessions/**`를 호출한다.
- 참가자 ID, server session/status/deadline, 단건 event ACK와 polling을 전제로 한다.
- `/v1/poc/scenario-catalog`, `/v1/poc/assessment`, `/v1/poc/snapshot` 호출 및 최종 client
  bundle 타입이 없다.
- 시나리오 내부 candle/news 표시와 UP/DOWN·확신도·근거 UI는 재사용할 수 있다.

이는 backend에 이전 endpoint를 되살려 해결하지 않는다. 프론트가 위 공개 API 네 개를 사용하고,
설문·세 scenario version/checksum·문항 timer·전역 sequence·최종 events를 브라우저 메모리에서
관리하도록 adapter/controller를 교체해야 한다. 공개 Pages는 현재 API base가 비어 demo mode로
빌드되므로 실제 연동 완료로 간주하지 않는다.
