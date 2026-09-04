# Safe-T · 기획·문서 저장소

> 서비스명: Safe-T(세이프티) · 「AI 레버리지 안전투자 훈련 플랫폼」
> Simulated AI Finance Education & Training
> 대회: 2026 금융 AI Challenge (주최 금융보안원 · 후원 금융위원회 · 운영 데이콘)
> 최종 마감: 2026-09-07(월) 10:00

2026 금융 AI Challenge 출품을 위한 기획서, 리서치, 설문, UI 설계 문서를 모아둔 팀 공용 저장소다.
구현 코드는 `feelgood-safe-t/backend`와 `feelgood-safe-t/frontend`에서 따로 관리한다.

## 먼저 알아둘 것

- 9/7 마감 산출물은 PPT가 아니라 기획서 PDF, 기능명세서 PDF, 배포 URL 세 가지다.
- 발표자료(PDF)는 상위 11팀에 들었을 때 10/8에 내고, 발표 심사는 10/13이다.
- 배포 URL은 9/7 11:00부터 9/11 23:59까지 접속되지 않으면 결격이다.
- 팀은 4인으로 확정했다(규정상 최대 4인). 리뷰 담당은 팀원이 아니므로 제출 서류의 구성원란에 적지 않는다.
- `spec/README.md`가 중심 기획안이다. 나머지 문서는 모두 여기에 맞춘다.

## 문서

| 경로 | 내용 |
|---|---|
| [`spec/README.md`](spec/README.md) | 중심 기획안(채택안). PoC 범위, 평가 배점, AI 역할 경계 |
| [`00-competition/competition-overview-and-checklist.md`](00-competition/competition-overview-and-checklist.md) | 규정, 일정, 심사기준, 제출물 체크리스트 (공식 페이지 원문 기반) |
| [`01-research/01-problem-definition-evidence.md`](01-research/01-problem-definition-evidence.md) | 레버리지 사태 타임라인, 모의거래 의무화 제도 분석, 금융이해력 통계, 출처 |
| [`01-research/02-competitive-analysis.md`](01-research/02-competitive-analysis.md) | 제도형·증권사형·앱형 3계층 비교, 공통 한계 C1~C5, 포지셔닝 |
| [`02-proposal/proposal-draft-v0.9.md`](02-proposal/proposal-draft-v0.9.md) | 메인 산출물. 공모전 기획서 초안이며 첨부1 양식 7개 항목에 1:1로 대응한다 |
| [`02-proposal/spec-gap-analysis.md`](02-proposal/spec-gap-analysis.md) | 중심 기획안 대조 분석. 충돌, 공백, 확인 요청, 문서별 조치 |
| [`02-proposal/scoring-guidance.md`](02-proposal/scoring-guidance.md) | 채점 가이던스 검토. 배점표가 곧 rubric임을 확인한 근거와 남은 전달 사항 |
| [`03-survey/01-demand-validation-survey.md`](03-survey/01-demand-validation-survey.md) | ⏸️ 보류. 이번 대회에서는 실시하지 않는다 |
| [`03-survey/02-investor-profiling-questionnaire.md`](03-survey/02-investor-profiling-questionnaire.md) | 온보딩 설문 확정 명세 v1.1. 문항 ID, 선택지 코드, 시나리오 매칭과 채점 연결 |
| [`03-survey/03-onboarding-questions-share.md`](03-survey/03-onboarding-questions-share.md) | ⭐ 공유용. 온보딩 10문항의 질문, 선택지, 의도만 정리 |
| [`04-uiux/user-flow-and-wireframes-v0.6.md`](04-uiux/user-flow-and-wireframes-v0.6.md) | ⏸️ 프론트엔드 전담. 구현 결과를 기능명세서로 옮기기 위한 참고 자료 |
| [`05-meetings/2026-09-02-meeting-agenda.md`](05-meetings/2026-09-02-meeting-agenda.md) | 9/2 미팅 결과, 9/6 안건, 결정사항, 마감 역산 일정 |
| [`06-implementation/repo-alignment.md`](06-implementation/repo-alignment.md) | 구현 저장소 참고 메모. 기획 문서를 사실에 맞게 쓰기 위한 대조 기록 |
| `99-templates/` | 대회 공식 hwpx 양식 2종 (첨부1 기획서, 첨부2 기능명세서) |

## 작성 규칙

- 경로와 파일명은 ASCII만 쓴다. 한글 디렉터리나 파일명은 OS 사이를 오가거나 클라우드에 동기화할 때 인코딩(NFC/NFD) 문제로 깨질 수 있다. 문서 내용은 한국어로 쓴다.
- 파일명은 kebab-case로 쓰고, 버전이 있는 문서에는 `-v0.9`처럼 접미사를 붙인다.
- 디렉터리 앞의 번호(`00-`부터 `99-`까지)는 정렬 순서를 고정하려는 것이므로 임의로 바꾸지 않는다.
- 상태 표기는 다음과 같다. 🔴 팀 결정 필요 · 🟡 근거나 수치 확보 필요 · ✅ 확정 · ⏸️ 보류

## 링크

- 대회 페이지: https://daker.ai/public/hackathons/2026-finance-ai-challenge
- 이 저장소: https://github.com/feelgood-safe-t/docs
