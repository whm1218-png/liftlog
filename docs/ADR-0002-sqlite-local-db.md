# ADR-0002: SQLite(로컬) + Firebase(클라우드) 하이브리드 데이터 저장

- 상태: 채택됨 (Accepted)
- 날짜: 2026-03-15
- 결정자: whm1218-png
- 관련 문서: docs/architecture.md, lessons/2026-05-18-sqlite-sync-flag-bug.md

## 컨텍스트 (배경)

LiftLog는 다음 두 조건을 동시에 만족해야 한다.

1. **헬스장은 인터넷이 자주 끊긴다.** 지하 1층·B2층에 위치한 곳이 많고, 와이파이가 없거나 통신사 신호가 약한 경우가 흔하다. 따라서 운동 기록은 **오프라인에서도 즉시 저장**되어야 한다.
2. **사용자가 폰을 바꾸거나 앱을 재설치해도 데이터가 보존**되어야 한다. 헬스 기록은 수개월~수년치의 자산이므로 영구 백업이 필요하다.

로컬 단독 저장(SQLite만 사용)은 조건 1은 만족하지만 2를 만족하지 못한다. 클라우드 단독 저장(Firestore만 사용)은 조건 2는 만족하지만 1을 만족하지 못한다.

## 검토한 대안

| 대안 | 오프라인 동작 | 데이터 영구 보관 | 비용 |
|------|--------------|------------------|------|
| SQLite 단독 | 가능 | 폰 변경 시 손실 | 무료 |
| Firestore 단독 | 불가 (오프라인 캐시 한계) | 가능 | Spark Plan 무료 한도 |
| **SQLite + Firestore 하이브리드** | 가능 | 가능 | Spark Plan 무료 한도 |
| Realm + MongoDB | 가능 | 가능 | 학습 비용 큼 |

## 결정

**SQLite(sqflite)로 로컬에 먼저 저장하고, 네트워크가 가능할 때 Firestore에 백업하는 하이브리드 구조를 채택한다.**

구조:
1. 사용자가 세트를 기록 → 즉시 SQLite에 INSERT (synced=0)
2. 백그라운드에서 synced=0 인 레코드를 Firestore에 업로드
3. 업로드 성공 후에만 synced=1 로 갱신
4. 다른 기기에서 로그인 시 Firestore → SQLite로 동기화

## 결과

- sqflite로 로컬 DB (`liftlog.db`) 관리
- Firebase Auth로 사용자 식별
- Cloud Firestore로 백업
- `synced` 플래그로 동기화 상태 추적 (lessons/2026-05-18 참조)

## 시행착오

초기 구현에서 `synced=1`을 업로드 전에 먼저 표시하는 버그가 있었다. 업로드가 실패해도 synced=1로 남아서 재시도가 되지 않았다. "업로드 성공 확인 → 그 후에 synced=1" 순서로 수정. 자세한 내용은 `lessons/2026-05-18-sqlite-sync-flag-bug.md` 참조.

## 참고

- sqflite: https://pub.dev/packages/sqflite
- Firestore: https://firebase.google.com/docs/firestore
