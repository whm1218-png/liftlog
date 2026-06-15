# BONUS.md — 가산점 신청 트래킹 (15주차 최종 기준)

> 발표 슬라이드 마지막 1장(8번)에서 어필. 너무 길게 설명하지 않기.

---

## 신청 현황

| 항목 | 가산점 | 신청 | 증빙 |
|------|--------|------|------|
| A. AI Agent/스킬/워크플로우 적극 활용 | +1 | ✅ | `.github/prompts/`, `.github/agents/`, `AGENTS.md` |
| B. 본인만의 기법 (단일 md 통합) | +2 | ✅ | `AUTHORING.[이름].md` |
| C. LLM Wiki 기반 본인 암묵지 운영 | +1 | ✅ | `lessons/` |
| D. AI Agent 리포트 발표 (별도 시간) | +2 | 🔲 | 별도 시간 신청 여부 확인 필요 |

**현재 확보 가산점: +4 (A+B+C)**

---

## A. AI Agent / 워크플로우 적극 활용 (+1) ✅

### 6단계 슬래시 명령 워크플로우
```
/spec → /plan → /implement → /test → /review → /retro
```
위치: `.github/prompts/*.prompt.md`

### 서브에이전트 3개
| 에이전트 | 역할 | 권한 |
|---------|------|------|
| `@code-reviewer` | 코드 리뷰 (읽기 전용) | 수정 금지 |
| `@test-writer` | 테스트 작성 | 읽기/쓰기 |
| `@bug-investigator` | 버그 원인 가설 제시 | 읽기 전용 |

### 정책 통합
`AGENTS.md` — AI Agent가 따라야 할 절대 규칙 6개 + 레이어 위반 금지 + 시크릿 보호 규칙 명문화

---

## B. 본인만의 기법 — 단일 md 부트스트랩 (+2) ✅

`AUTHORING.[이름].md` 하나로:
- 6단계 워크플로우 전체 설명
- 서브에이전트 카탈로그
- LiftLog 전용 고정 컨텍스트 (아키텍처, DB, 1RM 공식 규칙)
- AI 활용 금지 사항
- 새 환경 부트스트랩 절차 (5분 설치)

**발표 시 한 줄 설명**:
> "이 한 파일만 있으면 새 프로젝트에서도 동일한 AI Agent 워크플로우를 5분 안에 재현합니다."

---

## C. LLM Wiki 기반 암묵지 운영 (+1) ✅

`lessons/` 폴더에 디버깅·회고 기록:

- `2026-05-18-sqlite-sync-flag-bug.md` — Firebase 동기화 재업로드 버그 (원인/해결/재발방지 테스트)

각 항목 형식: 상황 → 재현절차 → 기대/실제결과 → 시도1(실패)/시도2(성공) → 진짜원인 → 다음에 적용할 점 → 추가된 테스트

---

## D. AI Agent 리포트 발표 (+2) 🔲

별도 시간 배정 여부 확인 필요. 후보 주제:
- Claude Code / Cursor / GitHub Copilot 실전 비교
- MCP(Model Context Protocol) 생태계 최신 동향

---

## 문서화 5점 체크 (과제 점수)

| 항목 | 점수 | 파일 |
|------|------|------|
| 기획서/요구사항 | +1 | `.planning/00-vision.md`, `01-requirements.md` |
| WBS/일정 | +1 | `.planning/02-wbs.md`, `04-schedule.md` |
| 아키텍처/ADR | +1 | `docs/architecture.md`, `.planning/decisions/ADR-0001~0003` |
| setup/deploy/testing | +1 | `docs/setup.md`, `docs/deploy.md`, `docs/testing.md` |
| AGENTS.md + README | +1 | `AGENTS.md`, `README.md` |

**문서화 점수: +5/5 (전체 충족)**

---

## GitHub 잔디 안내

> 채점은 `https://num.slogs.dev` 연결 GitHub 주소의 커밋 잔디로 확인됩니다.
> 10~15주차에 걸쳐 꾸준히 커밋되었는지 확인 — 한 번에 몰아서 push하지 말 것.

---

## 최종 발표 슬라이드 어필 스크립트 (8번 슬라이드)

> "AI 활용 측면에서는 spec부터 retro까지 6단계 워크플로우와 3개의 서브에이전트를 사용했고,
> AGENTS.md 한 파일에 모든 정책을 정리했습니다.
> 가산점은 AI 워크플로우 활용, AUTHORING.md 단일 부트스트랩, lessons 암묵지 운영을 신청합니다."
