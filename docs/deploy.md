# Deploy

> "배포는 어떻게 하셨나요?" — 이 문서로 답변합니다.

---

## 1. 빌드 종류

| 종류 | 명령 | 용도 |
|------|------|------|
| Debug | `flutter run --debug` | 개발 중 핫리로드 |
| Profile | `flutter run --profile` | 성능 측정 |
| Release | `flutter build apk --release` | 실제 배포용 |

---

## 2. Android 빌드

### APK (사이드로드 / Firebase App Distribution용)

```bash
flutter build apk --release --split-per-abi
```

산출물 위치:
```
build/app/outputs/flutter-apk/
├── app-armeabi-v7a-release.apk
├── app-arm64-v8a-release.apk
└── app-x86_64-release.apk
```

### AAB (Google Play 제출용)

```bash
flutter build appbundle --release
```

산출물:
```
build/app/outputs/bundle/release/app-release.aab
```

---

## 3. iOS 빌드

```bash
flutter build ios --release
open ios/Runner.xcworkspace
```

Xcode에서 **Product → Archive → Distribute App** 으로 TestFlight 또는 Ad Hoc 배포.

> iOS 빌드는 macOS 환경에서만 가능합니다.

---

## 4. 서명 / 인증서 관리

| 플랫폼 | 필요한 것 | 보관 위치 |
|--------|-----------|-----------|
| Android | keystore (`.jks`), `key.properties` | 로컬, **git 커밋 금지** |
| iOS | Apple Developer 계정, 인증서, Provisioning Profile | Xcode / Keychain |

**`android/key.properties` 예시** (커밋하지 않음):
```properties
storePassword=****
keyPassword=****
keyAlias=liftlog
storeFile=../liftlog-release.jks
```

`android/.gitignore`에 추가:
```
key.properties
*.jks
```

---

## 5. 환경 분리

```
.env.dev      # 로컬 개발 — 목 데이터, verbose 로깅
.env.staging  # 내부 테스트 — 실 Firebase, 디버그 가능
.env.prod     # 실배포 — 실 Firebase, 디버그 차단
```

**규칙**:
- `.env.example`만 git에 commit (키 값은 비워둠)
- 실제 `.env*`는 `.gitignore`에 포함
- Firebase API 키 등 시크릿은 **절대 코드에 하드코딩하지 않음**

`.env.example` 예시:
```
FIREBASE_API_KEY=
FIREBASE_PROJECT_ID=
FIREBASE_MESSAGING_SENDER_ID=
FIREBASE_APP_ID=
```

---

## 6. 현재 배포 채널 — Firebase App Distribution

본 프로젝트(7주 규모)의 배포 목표는 **"최소 APK 산출 + 1인 이상 설치 확인"** 입니다.

```bash
# 1. release APK 빌드
flutter build apk --release

# 2. Firebase CLI로 배포
firebase appdistribution:distribute \
  build/app/outputs/flutter-apk/app-release.apk \
  --app <FIREBASE_APP_ID> \
  --groups "testers" \
  --release-notes "v0.1.0 - 중간 데모 빌드"
```

테스터는 이메일로 받은 링크로 APK를 설치합니다.

---

## 7. 버전 관리 규칙 (SemVer)

```
MAJOR.MINOR.PATCH  예: 0.3.1

MAJOR — 호환되지 않는 큰 변경 (DB 스키마 대변경 등)
MINOR — 기능 추가 (Should Have 기능 출시)
PATCH — 버그 수정
```

`pubspec.yaml`:
```yaml
version: 0.3.1+4   # 0.3.1 = 버전명, +4 = 빌드 번호 (매 빌드 +1)
```

| 버전 | 시점 | 내용 |
|------|------|------|
| 0.1.0 | 12주차 | 핵심 기능(기록, 1RM, 차트) MVP |
| 0.2.0 | 13주차 | 휴식 타이머, 테스트 추가 |
| 0.3.0 | 14주차 | Firebase 백업, 배포 파이프라인 |
| 1.0.0 | 15주차 | 최종 발표 빌드 |

---

## 8. 롤백 방법

### Firebase App Distribution
이전 버전 APK를 동일 그룹에 재배포 (release notes에 "rollback" 명시)

### Google Play (사용 시)
Play Console → 프로덕션 → 이전 버전으로 **단계적 출시 일시중지** 후 이전 릴리스를 재출시

### 로컬 데이터 마이그레이션 롤백
`onDowngrade` 콜백은 기본적으로 구현하지 않음 — 대신 마이그레이션 전 자동 백업(DB 파일 복사)으로 대응 ([RISK-001](.planning/05-risks.md) 참고)

---

## 9. 배포 전 최종 체크리스트

- [ ] 버전 번호(`pubspec.yaml`) 업데이트
- [ ] 변경 로그(`CHANGELOG.md`) 작성
- [ ] `print()` / `debugPrint()` 디버그 로그 제거
- [ ] 디버그 전용 메뉴/버튼 제거
- [ ] `.env.prod` 값 확인 (Firebase 프로덕션 프로젝트 연결 여부)
- [ ] `flutter test` 전체 통과
- [ ] release APK 설치 후 핵심 플로우(기록→1RM→차트) 수동 확인

---

## 10. CI/CD — GitHub Actions

> `.github/workflows/build.yml` — 태그 push 시 자동 APK 빌드

```yaml
name: Build
on:
  push:
    tags: ['v*']
jobs:
  android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
      - run: flutter pub get
      - run: flutter test
      - run: flutter build apk --release
      - uses: actions/upload-artifact@v4
        with:
          name: app-release
          path: build/app/outputs/flutter-apk/
```

**사용법**:
```bash
git tag v0.3.0
git push origin v0.3.0
# → GitHub Actions가 자동으로 테스트 + APK 빌드 + 아티팩트 업로드
```
