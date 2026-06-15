# AUTHORING.[이름].v1.0.md

> LiftLog 프로젝트 전용 AI Agent 워크플로우 부트스트랩 파일  
> 이 파일 하나로 새 환경에서 동일한 워크플로우를 5분 안에 구축합니다.

---

## 한 줄 철학

> **"AI가 생성, 내가 결정."**  
> 코드와 문서는 AI가 초안을 만들고, 검토·수정·결정은 반드시 직접 한다.

---

## 6단계 워크플로우

```
/spec  →  /plan  →  /implement  →  /test  →  /review  →  /retro
사양      계획      구현           테스트    리뷰        회고
```

### 각 단계 설명

| 단계 | 슬래시 명령 | 산출물 | AI 역할 | 내 역할 |
|------|-----------|--------|---------|---------|
| 사양 | `/spec [기능명]` | `docs/specs/*.md` | 시나리오·수용조건 초안 | 요구사항 확인·수정 |
| 계획 | `/plan [사양파일]` | 작업 목록 | 작업 분해·우선순위 | 일정 현실성 검토 |
| 구현 | `/implement` | 코드 파일 | 레이어별 코드 초안 | 아키텍처 위반 검토 |
| 테스트 | `/test [대상]` | `test/**/*_test.dart` | AAA 패턴 테스트 초안 | 빠진 케이스 보강 |
| 리뷰 | `/review` | 리뷰 코멘트 | 코드 품질 체크리스트 | 수정 여부 최종 결정 |
| 회고 | `/retro` | `lessons/*.md` | KPT 회고 초안 | 다음 주 우선순위 결정 |

---

## 서브에이전트 카탈로그

| 에이전트 | 파일 | 언제 쓰는가 |
|---------|------|-----------|
| `@code-reviewer` | `.github/agents/code-reviewer.agent.md` | PR 전 자체 리뷰 |
| `@test-writer` | `.github/agents/test-writer.agent.md` | 테스트 커버리지 보강 |
| `@bug-investigator` | `.github/agents/bug-investigator.agent.md` | 원인 불명 버그 발생 시 |

---

## 부트스트랩 설치 절차

> 새 프로젝트에서 이 파일을 루트에 놓고 아래 명령을 실행:

```bash
# 1. 프롬프트 디렉토리 생성
mkdir -p .github/prompts .github/agents docs/specs lessons

# 2. 슬래시 명령 설치 확인
ls .github/prompts/
# spec.prompt.md  plan.prompt.md  implement.prompt.md
# test.prompt.md  review.prompt.md  retro.prompt.md

# 3. 에이전트 설치 확인
ls .github/agents/
# code-reviewer.agent.md  test-writer.agent.md  bug-investigator.agent.md

# 4. 워크플로우 시작
# /spec [첫 번째 기능명] 으로 시작
```

---

## LiftLog 전용 컨텍스트 규칙

AI에게 항상 함께 전달하는 고정 컨텍스트:

```
프로젝트: LiftLog — Flutter 헬스 트래킹 앱
아키텍처: 4레이어 MVVM (Presentation / Application / Domain / Data)
DB: SQLite (sqflite) + Firebase Firestore 백업
상태관리: Riverpod
1RM 공식: Epley (weight × (1 + reps/30))
핵심 규칙: UI에서 SQLite 직접 접근 금지, 반드시 Repository 경유
```

---

## AI 활용 시 금지 사항

- ❌ AI 생성 코드를 읽지 않고 그대로 커밋
- ❌ 마이그레이션 코드를 AI에게 검토 없이 배포
- ❌ "고쳐줘" 만 말하고 원인 파악 없이 수정 수용
- ❌ 테스트 없이 핵심 로직 변경

---

## 발표 시 어필 포인트

> "저는 이 한 파일(`AUTHORING.[이름].md`)만 있으면  
>  새 프로젝트에서도 동일한 AI Agent 환경을 5분 안에 구축할 수 있습니다.  
>  6단계 슬래시 명령과 3개의 서브에이전트로  
>  기획부터 테스트까지 일관된 워크플로우를 유지했습니다."

---

## 버전 히스토리

| 버전 | 날짜 | 변경 내용 |
|------|------|---------|
| v1.0 | 2026-05-18 | 최초 작성 — 6단계 워크플로우 + 3 에이전트 |
