# 2026-05-18 — SQLite synced 플래그 업데이트 누락

## 상황

Firebase 동기화 후에도 `synced = 0` 레코드가 계속 재업로드됨.
앱을 켤 때마다 동일한 세트 기록이 Firestore에 중복 저장되는 버그 발견.

## 재현 절차

1. 오프라인 상태에서 세트 3개 기록 (synced = 0으로 저장됨)
2. Wi-Fi 연결
3. 앱 백그라운드 → 포그라운드 전환
4. Firestore 콘솔에서 동일 문서가 3개 → 6개 → 9개로 증가 확인

## 기대 결과

업로드 성공 후 로컬 DB의 `synced = 1`로 업데이트되어 재업로드 안 됨

## 실제 결과

`synced` 값이 여전히 0으로 남아 있어 매번 재업로드 발생

## 시도 1 — 실패

업로드 함수 내부에 `synced = 1` 업데이트 코드가 있다고 가정하고
Firestore 배치 크기 제한 문제를 먼저 확인 → 관련 없음

## 시도 2 — 성공

`FirestoreSyncService.uploadPending()` 코드를 다시 읽어보니
Firestore `batch.commit()` **이후**가 아닌 **이전**에 `synced` 업데이트 쿼리를 실행하고 있었음.
commit이 실패해도 synced가 1이 되는 문제 + 순서 자체도 잘못됨.

```dart
// 버그 코드
await db.update('sets', {'synced': 1}, where: 'synced = 0'); // 먼저 실행
await batch.commit(); // 그 다음 업로드

// 수정 코드
await batch.commit(); // 업로드 먼저
await db.update('sets', {'synced': 1}, where: 'id IN (${ids.join(',')})'); // 성공 후 업데이트
```

## 진짜 원인

트랜잭션 순서 오류. 업로드 성공 확인 전에 로컬 상태를 변경한 것이
아니라, 반대로 로컬 상태를 먼저 바꾼 뒤 업로드를 시도한 구조였음.

## 다음에 비슷한 문제 만났을 때

- 동기화 로직은 항상 **"원격 작업 성공 확인 → 로컬 상태 변경"** 순서
- `commit()` 반환값(Future)을 `await` 없이 사용했는지 확인
- 업로드 후 반드시 회귀 테스트: 오프라인 기록 → 온라인 전환 시나리오

## 추가된 테스트

```dart
test('should_not_re-upload_when_already_synced', () async {
  // 업로드 성공 후 synced = 1인지 확인
  await syncService.uploadPending();
  final pending = await db.query('sets', where: 'synced = 0');
  expect(pending, isEmpty);
});
```
