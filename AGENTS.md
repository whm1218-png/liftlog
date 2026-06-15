# AGENTS.md

> 이 파일은 AI Agent(Cursor, Claude Code 등)가 LiftLog 프로젝트에서 작업할 때 따라야 할 규칙을 정의합니다.  
> 사람 개발자에게도 동일하게 적용되는 프로젝트 컨벤션입니다.

---

## 프로젝트 개요

- **이름**: LiftLog
- **목적**: 점진적 과부하 기록 + 1RM 자동 계산이 통합된 헬스 전용 모바일 앱
- **스택**: Flutter · SQLite(sqflite) · Firebase(Auth/Firestore) · Riverpod
- **아키텍처**: 4레이어 MVVM (Presentation / Application / Domain / Data) — 자세한 내용은 [docs/architecture.md](docs/architecture.md)

---

## 절대 규칙

1. **레이어 위반 금지**
   UI(Presentation)에서 SQLite/Firestore에 직접 접근하지 않는다. 반드시 `Repository` 인터페이스를 경유한다.

2. **Domain은 순수 Dart**
   `domain/` 폴더는 Flutter, sqflite, Firebase를 import하지 않는다. (Epley 공식 등 핵심 로직은 프레임워크 독립적이어야 함)

3. **1RM 계산은 Epley 공식만 사용**
   `weight × (1 + reps / 30)`. 변경 시 [ADR-0003](.planning/decisions/ADR-0003-epley-1rm-formula.md) 갱신 필요.

4. **DB 마이그레이션은 항상 추가만**
   `DROP TABLE`, 컬럼 삭제 금지. `ALTER TABLE ADD COLUMN`만 사용하고 `onUpgrade` 버전별 스크립트로 관리.

5. **시크릿 절대 커밋 금지**
   `.env`, `google-services.json`, `GoogleService-Info.plist`, `*.jks`는 항상 `.gitignore`. `.env.example`만 commit.

6. **테스트 동반**
   `domain/`, `data/` 레이어 변경 시 대응하는 테스트(`test/`)를 함께 작성/수정한다. AAA 패턴, `should_[기대결과]_when_[조건]` 명명.

---

## 코드 작성 규칙

- 새 화면 → `lib/presentation/screens/`
- 재사용 위젯 → `lib/presentation/widgets/`
- 화면 상태/로직 → `lib/application/view_models/` (Riverpod Provider)
- 비즈니스 규칙 → `lib/domain/services/`
- 데이터 모델 → `lib/domain/entities/`
- DB 쿼리 → `lib/data/local/`
- API/Firebase 호출 → `lib/data/remote/`

**판단 기준**: "이 코드가 Firebase 없이도 테스트될 수 있는가?"
→ Yes → `domain/` 또는 `application/` / No → `data/`

---

## AI Agent 워크플로우

이 프로젝트는 6단계 슬래시 명령 워크플로우를 사용한다 ([AUTHORING.[이름].md](AUTHORING.[이름].md) 참고):

```
/spec → /plan → /implement → /test → /review → /retro
```

| 명령 | 위치 |
|------|------|
| `/spec` | `.github/prompts/spec.prompt.md` |
| `/plan` | `.github/prompts/plan.prompt.md` |
| `/implement` | `.github/prompts/implement.prompt.md` |
| `/test` | `.github/prompts/test.prompt.md` |
| `/review` | `.github/prompts/review.prompt.md` |
| `/retro` | `.github/prompts/retro.prompt.md` |

### 서브에이전트

| 에이전트 | 위치 | 용도 | 권한 |
|---------|------|------|------|
| `@code-reviewer` | `.github/agents/code-reviewer.agent.md` | 코드 리뷰 (읽기 전용) | 수정 금지 |
| `@test-writer` | `.github/agents/test-writer.agent.md` | 테스트 작성 | 읽기/쓰기 |
| `@bug-investigator` | `.github/agents/bug-investigator.agent.md` | 버그 원인 가설 제시 (수정 금지) | 읽기 전용 |

---

## AI 생성 코드 검토 원칙

> "AI가 생성, 내가 결정."

- AI가 생성한 코드는 머지 전 반드시 사람이 읽고 이해한다.
- 마이그레이션, 동기화, 1RM 계산 등 핵심 로직은 AI 생성 후 **반드시 단위 테스트로 검증**한다.
- 디버깅 사례는 `lessons/`에 기록한다 (예: [2026-05-18-sqlite-sync-flag-bug.md](lessons/2026-05-18-sqlite-sync-flag-bug.md)).

---

## 커밋 / PR 규칙

- 커밋 메시지: `feat:`, `fix:`, `docs:`, `test:`, `chore:` 접두어 사용
- 버전: [Semantic Versioning](https://semver.org/lang/ko/) — `docs/deploy.md` 참고
- 태그 push(`v*`) 시 `.github/workflows/build.yml`이 테스트 + APK 빌드 자동 실행

---

## 참고 문서 지도

| 문서 | 내용 |
|------|------|
| [README.md](README.md) | 프로젝트 5분 소개 |
| [docs/setup.md](docs/setup.md) | 개발 환경 설정 |
| [docs/architecture.md](docs/architecture.md) | 아키텍처 다이어그램 + 레이어 |
| [docs/deploy.md](docs/deploy.md) | 빌드/배포/버전관리 |
| [docs/testing.md](docs/testing.md) | 테스트 실행 가이드 |
| [docs/security-checklist.md](docs/security-checklist.md) | 보안 체크리스트 |
| [.planning/decisions/](.planning/decisions/) | ADR-0001~0003 |
| [.planning/05-risks.md](.planning/05-risks.md) | 위험 분석 |
| [lessons/](lessons/) | 디버깅·회고 기록 |
