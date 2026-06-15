# Security Checklist

> OWASP Mobile Top 10을 LiftLog 규모에 맞게 압축한 체크리스트.  
> "보안은 어떻게 챙기셨나요?" Q&A 대응 문서.

---

## 적용 현황 요약

| 영역 | 상태 | 비고 |
|------|------|------|
| 1. 비밀 관리 | ✅ 적용 | `.env` + `.gitignore` |
| 2. 입력 검증 | ✅ 적용 | 무게·반복 입력 범위 검증 |
| 3. 통신 | ✅ 적용 | Firebase SDK 기본 HTTPS |
| 4. 로컬 저장 | ✅ 적용 | flutter_secure_storage |
| 5. 로그 | 🔶 진행중 | release 빌드 로그 차단 설정 중 |
| 6. 권한 | ✅ 적용 | 알림 권한만 사용, 사유 명시 |
| 7. 의존성 | 🔶 진행중 | `flutter pub outdated` 정기 확인 예정 |
| 8. 앱 자체 | ⬜ v2 검토 | 스냅샷 보호는 범위 외 |

> **발표 Q&A 답변 포인트**: 위 8개 중 **1, 3, 4번**을 가장 또렷이 설명한다.

---

## 1. 비밀 관리

- [x] Firebase API 키는 `.env`에서 로드, 코드에 하드코딩하지 않음
- [x] `.env`, `android/key.properties`, `*.jks`, `*.p12`, `*.mobileprovision` 모두 `.gitignore`
- [x] `.env.example`만 git에 commit (값은 비워둠)
- [ ] git history 노출 점검: `git log -p | grep -i -E "(api_key|secret|password)"`

```gitignore
# .gitignore
.env
.env.*
!.env.example
android/key.properties
*.jks
*.keystore
ios/Runner/GoogleService-Info.plist
android/app/google-services.json
```

> `google-services.json` / `GoogleService-Info.plist`도 비공개 — 각자 Firebase 콘솔에서 다운로드해 로컬에 배치 (`docs/setup.md` 참고)

---

## 2. 입력 검증

세트 기록 입력값에 대한 검증 규칙:

| 입력 | 검증 |
|------|------|
| 무게 (weight) | `0 < weight ≤ 500` (kg), 숫자만 |
| 반복 (reps) | `0 < reps ≤ 100`, 정수만 |
| 운동명 (커스텀) | 길이 1~30자, 특수문자 제한 |

```dart
// domain/services/one_rep_max_service.dart
if (reps <= 0) throw ArgumentError('reps must be positive');
if (weight <= 0) throw ArgumentError('weight must be positive');
```

- [x] SQL Injection — sqflite의 파라미터 바인딩(`?`) 사용, 문자열 concat 금지
- [x] 경로 탈출 — 파일 시스템 직접 접근 없음 (sqflite/Firestore만 사용)

---

## 3. 통신

- [x] Firebase SDK는 기본적으로 HTTPS만 사용 (별도 설정 불필요)
- [x] 인증서 검증 비활성화 코드 없음 (`badCertificateCallback` 등 미사용)
- [x] 인증 토큰은 Firebase Auth SDK가 헤더로 자동 첨부, URL 쿼리에 노출 안 됨
- [x] 토큰 만료 시 Firebase Auth가 자동 갱신 (`idTokenChanges` 리스너)

---

## 4. 로컬 저장

| 데이터 | 저장 위치 | 암호화 |
|--------|-----------|--------|
| 운동 기록 (무게·반복) | SQLite | 평문 (민감정보 아님) |
| Firebase Auth 토큰 | `flutter_secure_storage` | OS Keychain/Keystore |
| 사용자 이메일 | Firebase Auth 내부 | SDK 관리 |

- [x] 비밀번호는 앱에서 직접 저장하지 않음 (Firebase Auth가 처리)
- [x] 토큰은 `flutter_secure_storage` 사용 (iOS Keychain / Android EncryptedSharedPreferences 기반)

---

## 5. 로그

- [ ] release 빌드에서 `print()` / `debugPrint()` 제거 또는 `kReleaseMode` 가드 적용

```dart
if (!kReleaseMode) {
  debugPrint('sync result: $result');
}
```

- [x] 비밀번호·토큰·PII를 로그에 출력하지 않음 (코드 리뷰 체크리스트 항목, `.github/agents/code-reviewer.agent.md` 참고)

---

## 6. 권한

| 권한 | 사용처 | 사유 |
|------|--------|------|
| 알림 (Notification) | 휴식 타이머 완료 알림 | "휴식 시간이 끝나면 알려드립니다" 안내 문구 표시 |

- [x] 카메라, 위치, 연락처 등 사용하지 않는 권한은 요청하지 않음
- [x] 알림 권한 거부 시 — 타이머는 화면 내 시각적 표시로 동작 (graceful fallback)

---

## 7. 의존성

```bash
# 정기 점검 명령
flutter pub outdated
```

- [ ] 주요 패키지(sqflite, firebase_*, riverpod) 최신 버전 확인 — 14주차 진행 중
- [x] 모두 pub.dev에서 활발히 유지보수되는 패키지 (주간 다운로드 1만+ 기준)

---

## 8. 앱 자체 (v2 검토)

- [ ] 백그라운드 진입 시 운동 기록 화면 가림 (스냅샷 보호) — 헬스 데이터는 민감도 낮아 v1 범위 외
- [ ] 루팅/탈옥 탐지 — v1 범위 외
- [ ] 디버거 부착 탐지 — v1 범위 외

> 운동 기록 데이터는 금융·의료 정보 대비 민감도가 낮아, 8번 항목은 v2에서 사용자 요청 시 검토

---

## Q&A 대응 스크립트

> "보안은 어떻게 챙기셨나요?"

"세 가지를 챙겼습니다. 첫째, Firebase API 키는 `.env` 파일로 분리하고 `.gitignore`에 포함시켜 git에 노출되지 않도록 했습니다. 둘째, 로그인 토큰은 `flutter_secure_storage`를 사용해 OS의 Keychain/Keystore에 암호화 저장합니다. 셋째, SQLite 쿼리는 파라미터 바인딩을 사용해 SQL Injection을 방지했습니다."
