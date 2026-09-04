# 구현 저장소 정합성 기록

> 갱신 2026-09-04 · 기준: `spec/README.md`, Backend Engine runtime `0.4.0`

## 결론

Backend Engine은 중앙 엔진의 PoC 범위인 시나리오 재생, 설문·단일 UP/DOWN·뉴스 열람 기록,
불변 Evaluation Snapshot 제공을 구현한다. LLM 호출, 성향 해석, 시나리오 추천, 점수·PASS와
보고서는 엔진 밖에 둔다.

## 일치하는 계약

| 영역 | 제품 계약과 구현 |
|---|---|
| 시험 | 서로 다른 단일 자산 문항 정확히 3개, 순차 진행 |
| 시간 | 문항당 180초, 시장 60배속, 서버 시각 권위 |
| 응답 | 문항당 UP/DOWN 정확히 1회, 변경·덮어쓰기 없음 |
| 응답 근거 | `reasonTags` 1개 이상과 `confidence` 필수, `reasonText` 선택 |
| 설문 | `questionnaire-safe-t-v2`, 10문항, 질문·선택지 상세와 선택 범위 고정 |
| 열람 | 뉴스 목록 제공과 상세 열람 이벤트를 구분하고 서버 시각·시장 offset 기록 |
| Snapshot | 설문 원본, 단일 응답, 뉴스 공개·열람 상태, 응답 전후 캔들 문맥을 불변 봉인 |
| 평가 경계 | 방향 적중·수익률을 계산하지 않으며 점수·PASS는 후단 책임 |
| 제외 범위 | 주문·포지션·비중·잔고·PnL·MDD·quiz 없음 |

## 의도적으로 남은 PoC 예외

### 현재 item 데이터 일괄 전달

현재 item의 캔들 240개와 뉴스 전체를 참가자 API 한 응답에 싣고 프론트가 offset에 따라
표시한다. 미래 item은 계속 잠근다. 이는 PoC API 단순화를 위한 결정이며, 현재 item 배열의
개발자 도구 선조회까지 막는 운영용 streaming/delta 전달은 후속 작업이다.

### raw mock 시나리오

가격 정규화·식별자 가명화·뉴스 재작성 파이프라인이 아직 없으므로 역사 자료 10개를
`mockRawSource=true`, `participantSafe=false`로 명시해 개발용으로 제공한다. 실제 참가자 배포
전에는 검수된 가명·정규화 package로 교체해야 한다.

## 저장소 간 전달 사항

- 프론트는 questionnaire 문구를 중복 관리하기보다 `GET /v1/questionnaires/current`의
  `questionnaire-safe-t-v2`를 소비하는 것이 기준이다.
- 프론트의 과거 3자산 자유 이동·방향 정오 채점·로컬 PASS 구현은 현재 제품 계약이 아니다.
- Selection Orchestrator는 설문을 해석하되 엔진 DB에 성향 등급이나 위험 한도를 쓰지 않는다.
- Evaluation Service는 Snapshot의 당시 공개 정보와 열람 사실만 사용하고 방향 적중률을
  산출하지 않는다.
