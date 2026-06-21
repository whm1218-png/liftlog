# ADR-0001: Flutter 프레임워크 선택

- 상태: 채택됨 (Accepted)
- 날짜: 2026-03-10
- 결정자: whm1218-png
- 관련 문서: docs/architecture.md

## 컨텍스트 (배경)

LiftLog는 헬스장에서 즉시 운동을 기록해야 하는 모바일 앱이다. 1인 개발 프로젝트이고 iOS와 Android 두 플랫폼을 모두 지원해야 한다. 따라서 다음 요구사항을 만족하는 프레임워크가 필요했다.

1. iOS·Android 단일 코드베이스
2. 빠른 개발 속도(핫 리로드)
3. 풍부한 위젯/UI 컴포넌트
4. 오프라인 동작 지원
5. SQLite·Firebase 같은 표준 라이브러리 지원

## 검토한 대안

| 대안 | 장점 | 단점 |
|------|------|------|
| **Flutter** | 단일 코드베이스, 핫 리로드, 위젯 일관성 높음, Material/Cupertino 모두 지원 | Dart 학습 곡선, 앱 크기 다소 큼 |
| React Native | JavaScript 친숙, 생태계 큼 | 네이티브 모듈 분기 필요, 일관성 낮음 |
| 네이티브 (Swift + Kotlin) | 성능 최상, 플랫폼 API 완전 활용 | 1인 개발에 코드베이스 2배, 시간 부족 |
| Ionic / Capacitor | 웹 기술 활용 가능 | 성능 한계, 운동 입력 UX에 부적합 |

## 결정

**Flutter를 선택한다.**

이유:
1. 1인 프로젝트에서 iOS·Android 동시 지원이 가능
2. 핫 리로드로 UI 반복 개발 속도가 빠름
3. 위젯 시스템이 일관적이라 디자인 일관성 확보가 쉬움
4. sqflite, firebase_core, riverpod 등 필요한 패키지가 모두 안정적으로 제공됨

## 결과

- pubspec.yaml에 Flutter SDK 3.4 이상으로 고정
- Material 3 디자인 시스템 사용
- 빌드 결과물: Android APK / iOS IPA

## 참고

- Flutter 공식 문서: https://docs.flutter.dev
- pubspec.yaml 참고
