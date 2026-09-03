# Safe-T — 기획·문서 레포

> **서비스명:** **Safe-T** (세이프티) · 「AI 레버리지 안전투자 훈련 플랫폼」
> Simulated AI Finance Education & Training
> **대회:** 2026 금융 AI Challenge (주최 금융보안원 · 후원 금융위원회 · 운영 데이콘)
> **최종 마감:** 🔴 **2026-09-07(월) 10:00**

2026 금융 AI Challenge 출품을 위한 **기획서·리서치·설문·UI 설계 문서**를 모아둔 팀 공용 레포입니다.
구현 코드는 별도 레포에서 관리합니다.

## 📌 지금 알아야 할 것

- 9/7 마감 산출물은 **PPT가 아니라 ① 기획서 PDF ② 기능명세서 PDF ③ 배포 URL** 3종이다.
- 발표자료(PDF)는 **상위 11팀 진출 시 10/8 마감**, 발표는 10/13.
- 배포 URL은 **9/7 11:00 ~ 9/11 23:59 접속 불가 시 결격**.
- 팀 **4인 확정** (규정상 최대 4인 — 충족). 리뷰 담당은 팀원이 아니므로 **제출 서류의 구성원란에 기재하지 않는다.**
- **`spec/README.md`가 중심 기획안이다.** 다른 모든 문서는 여기에 정렬한다.

## 📂 문서

| 경로 | 내용 |
|---|---|
| [`00-competition/competition-overview-and-checklist.md`](00-competition/competition-overview-and-checklist.md) | 규정·일정·심사기준·제출물 체크리스트 (공식 API 원문 기반) |
| [`01-research/01-problem-definition-evidence.md`](01-research/01-problem-definition-evidence.md) | 레버리지 사태 타임라인, 모의거래 의무화 제도 분석, 금융이해력 통계, 출처 |
| [`01-research/02-competitive-analysis.md`](01-research/02-competitive-analysis.md) | 제도형/증권사형/앱형 3계층 비교, 공통 한계 C1~C5, 포지셔닝 |
| [`spec/README.md`](spec/README.md) | **중심 기획안 (채택안).** PoC 범위·평가 배점·AI 역할 경계 |
| [`02-proposal/spec-gap-analysis.md`](02-proposal/spec-gap-analysis.md) | **중심 기획안 대조 분석.** 충돌·공백·확인 요청·문서별 액션 |
| [`02-proposal/proposal-draft-v0.8.md`](02-proposal/proposal-draft-v0.8.md) | **메인 산출물.** 스펙 반영본. 첨부1 양식 7개 항목에 1:1 매핑 |
| [`03-survey/01-demand-validation-survey.md`](03-survey/01-demand-validation-survey.md) | ⏸️ **보류** — 이번 대회에서는 실시하지 않음 |
| [`03-survey/02-investor-profiling-questionnaire.md`](03-survey/02-investor-profiling-questionnaire.md) | **확정 명세 v1.1.** 프론트 10문항 기준 + 시나리오 매칭·채점 연결 정의 + 백엔드 반영 과제 |
| [`04-uiux/user-flow-and-wireframes-v0.6.md`](04-uiux/user-flow-and-wireframes-v0.6.md) | ⏸️ **프론트엔드 전담.** 구현 결과를 기능명세서로 옮기기 위한 참고 자료 |
| [`06-implementation/repo-alignment.md`](06-implementation/repo-alignment.md) | **구현 저장소 참고 메모.** 기획 문서를 사실에 맞게 쓰기 위한 대조 기록 (backend·frontend는 관리 범위 밖) |
| [`05-meetings/2026-09-02-meeting-agenda.md`](05-meetings/2026-09-02-meeting-agenda.md) | 9/2 미팅 아젠다 · 결정사항 D1~D6 · 팀장 확인 요청 · 마감 역산 일정 |
| `99-templates/` | 대회 공식 hwpx 양식 2종 (첨부1 기획서 / 첨부2 기능명세서) |

## ✍️ 작성 규칙

- **경로·파일명은 ASCII만 사용한다.** 한글 디렉토리/파일명은 OS 간 이동이나 클라우드 동기화 시 인코딩(NFC/NFD) 문제로 깨질 수 있다. 문서 *내용*은 한국어로 쓴다.
- 파일명은 `kebab-case`, 버전이 있는 문서는 `-v0.6` 처럼 접미사를 붙인다.
- 디렉토리 접두 번호(`00-` ~ `99-`)는 정렬 순서를 고정하기 위한 것이므로 임의로 바꾸지 않는다.
- 문서 안의 상태 표기: 🔴 팀 결정 필요 · 🟡 근거·수치 확보 필요 · ✅ 확정

## 🔗 링크
- 대회 페이지: https://daker.ai/public/hackathons/2026-finance-ai-challenge
- 기획·문서 레포(이 저장소): https://github.com/feelgood-safe-t/docs
