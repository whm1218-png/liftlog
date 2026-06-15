# Setup

> 이 문서만 보고 5분 안에 앱을 실행할 수 있어야 합니다.  
> 막히는 부분은 아래 **자주 묻는 문제** 섹션을 먼저 확인하세요.

---

## 1. 사전 요구 사항

| 도구 | 최소 버전 | 확인 명령 |
|------|----------|-----------|
| Git | 2.40+ | `git --version` |
| Flutter SDK | 3.22+ | `flutter --version` |
| Dart | 3.4+ | `dart --version` (Flutter에 포함) |
| Android Studio | 2023.3+ | — (Android 에뮬레이터 필요 시) |
| Xcode | 15+ | `xcode-select --version` (iOS 빌드 시, macOS 전용) |
| Firebase CLI | 13+ | `firebase --version` |
| Node.js | 20.x | `node -v` (Firebase CLI 설치 용도) |

> iOS 빌드는 **macOS 환경만** 가능합니다. Windows/Linux는 Android 에뮬레이터로 진행하세요.

---

## 2. Flutter SDK 설치

### Windows

```powershell
# Windows — winget 사용
winget install Google.Flutter

# 또는 공식 사이트에서 zip 다운로드 후 PATH 추가
# https://docs.flutter.dev/get-started/install/windows
```

### macOS

```bash
# Homebrew 사용
brew install --cask flutter

# 설치 확인
flutter doctor
```

### Linux (Ubuntu / Debian)

```bash
# snap 사용
sudo snap install flutter --classic

# 설치 확인
flutter doctor
```

---

## 3. 저장소 클론

```bash
git clone https://github.com/[your-username]/[repo-name].git
cd [repo-name]
```

---

## 4. 의존성 설치

```bash
# Flutter 패키지 설치
flutter pub get
```

설치되는 주요 패키지:

| 패키지 | 버전 | 용도 |
|--------|------|------|
| `sqflite` | ^2.3.3 | 로컬 SQLite DB |
| `firebase_core` | ^3.3.0 | Firebase 초기화 |
| `firebase_auth` | ^5.1.4 | 이메일/Google 로그인 |
| `cloud_firestore` | ^5.2.1 | 클라우드 백업 |
| `riverpod` | ^2.5.1 | 상태 관리 |
| `go_router` | ^14.2.0 | 화면 라우팅 |
| `fl_chart` | ^0.69.0 | 진행 그래프 |
| `flutter_secure_storage` | ^9.2.2 | 인증 토큰 안전 저장 |

---

## 5. 환경 변수 설정 (Firebase)

```bash
# .env.example 복사
cp .env.example .env
```

`.env` 파일 내용 (Firebase 콘솔에서 발급):

```
FIREBASE_API_KEY=여기에_입력
FIREBASE_PROJECT_ID=여기에_입력
FIREBASE_MESSAGING_SENDER_ID=여기에_입력
FIREBASE_APP_ID=여기에_입력
```

> Firebase 콘솔 → 프로젝트 설정 → 앱 추가 → `google-services.json`(Android) / `GoogleService-Info.plist`(iOS) 다운로드 후 각각 아래 경로에 배치:
> - Android: `android/app/google-services.json`
> - iOS: `ios/Runner/GoogleService-Info.plist`

---

## 6. 빌드 확인

```bash
# 연결된 디바이스 / 실행 가능한 에뮬레이터 확인
flutter devices

# Android 에뮬레이터 실행 (Android Studio 먼저 열기)
flutter emulators --launch [emulator_id]

# 앱 실행
flutter run
```

**성공 시**: 시뮬레이터 또는 에뮬레이터에 로그인 화면이 표시됩니다.

---

## 7. 테스트 실행

```bash
# 전체 단위 테스트
flutter test

# 특정 파일만
flutter test test/domain/one_rep_max_test.dart
```

---

## 8. 자주 묻는 문제

### Q1. `flutter doctor` 에서 경고가 나와요
```
Doctor summary (to see all details, run flutter doctor -v)
[✓] Flutter ...
[!] Android toolchain ...
```
→ `[!]` 항목을 클릭해 나오는 설치 안내를 따라 해결합니다.  
→ Android 에뮬레이터만 사용하는 경우 Xcode 경고는 무시해도 됩니다.

### Q2. `flutter pub get` 실패 — 네트워크 오류
```
Error: Failed to retrieve package ... (host lookup failure)
```
→ VPN이 켜져 있다면 끄고 재시도합니다.  
→ `flutter pub cache repair` 실행 후 재시도합니다.

### Q3. 에뮬레이터가 실행되지 않아요
→ Android Studio → Device Manager → 에뮬레이터 생성 (API 34 이상 권장)  
→ BIOS에서 가상화(VT-x / AMD-V) 활성화 여부 확인 (Windows)

### Q4. iOS 빌드 시 인증서 오류
```
error: No signing certificate "iOS Development" found
```
→ Xcode → Preferences → Accounts → Apple ID 로그인  
→ `ios/` 폴더에서 `open Runner.xcworkspace` → Signing 탭 → Team 선택

### Q5. Firebase `google-services.json` 파일이 없다는 오류
```
FAILURE: Could not find google-services.json
```
→ Firebase 콘솔 → 프로젝트 설정 → 안드로이드 앱 → 구성 파일 다운로드  
→ `android/app/` 폴더에 파일 배치 후 재빌드

---

## 9. 폴더 구조 요약

```
lib/
├── main.dart              # 앱 진입점
├── app.dart               # 라우팅·테마 설정
├── presentation/          # 화면, 위젯, 테마
│   ├── screens/
│   ├── widgets/
│   └── theme/
├── application/           # ViewModel, UseCase, 상태
│   └── view_models/
├── domain/                # 핵심 비즈니스 규칙, 엔티티
│   ├── entities/
│   └── services/
└── data/                  # DB, API, Repository
    ├── repositories/
    ├── local/             # SQLite
    └── remote/            # Firebase
```

자세한 설명은 [`docs/architecture.md`](./architecture.md)를 참고하세요.
