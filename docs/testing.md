# Testing

> 이 문서만 보고 테스트를 실행할 수 있어야 합니다.

---

## 테스트 실행 명령

```bash
# 전체 단위 테스트
flutter test

# 특정 파일만
flutter test test/domain/one_rep_max_test.dart

# 커버리지 포함
flutter test --coverage

# 통합 테스트 (에뮬레이터 필요)
flutter test integration_test/workout_flow_test.dart
```

---

## 테스트 구조

```
test/
├── domain/
│   └── one_rep_max_test.dart       # Epley 공식 단위 테스트
├── data/
│   ├── migration_test.dart         # SQLite 마이그레이션 테스트
│   └── workout_repository_test.dart
├── application/
│   └── workout_vm_test.dart        # ViewModel 단위 테스트
└── widget/
    └── set_card_test.dart          # 위젯 테스트

integration_test/
└── workout_flow_test.dart          # 세트 기록 → 1RM → 저장 E2E
```

---

## 1. 단위 테스트 — Epley 1RM 공식

> 파일: `test/domain/one_rep_max_test.dart`

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:liftlog/domain/services/one_rep_max_service.dart';

void main() {
  final service = OneRepMaxService();

  group('OneRepMaxService', () {

    // ── Happy Path ──
    test('should_return_weight_when_reps_is_1', () {
      // Arrange
      const weight = 100.0;
      const reps = 1;
      // Act
      final result = service.estimate(weight, reps);
      // Assert
      expect(result, equals(100.0));
    });

    test('should_calculate_correctly_when_standard_input', () {
      // 80kg × 8회 → 80 × (1 + 8/30) = 93.33...
      final result = service.estimate(80.0, 8);
      expect(result, closeTo(93.33, 0.01));
    });

    // ── Edge Cases ──
    test('should_throw_when_reps_is_zero', () {
      expect(() => service.estimate(80.0, 0), throwsArgumentError);
    });

    test('should_throw_when_reps_is_negative', () {
      expect(() => service.estimate(80.0, -1), throwsArgumentError);
    });

    test('should_return_warning_flag_when_reps_exceeds_12', () {
      final result = service.estimateWithMeta(80.0, 15);
      expect(result.isHighRepWarning, isTrue);
    });
  });
}
```

**실행**:
```bash
flutter test test/domain/one_rep_max_test.dart
```

---

## 2. 단위 테스트 — SQLite 마이그레이션

> 파일: `test/data/migration_test.dart`

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';
import 'package:liftlog/data/local/database_helper.dart';

void main() {
  setUpAll(() {
    sqfliteFfiInit();
    databaseFactory = databaseFactoryFfi; // 인메모리 DB
  });

  test('should_preserve_data_when_migrating_v1_to_v2', () async {
    // Arrange: v1 DB 생성 + 데이터 삽입
    final db = await openDatabase(
      inMemoryDatabasePath,
      version: 1,
      onCreate: (db, v) async {
        await db.execute('''
          CREATE TABLE sets (
            id TEXT PRIMARY KEY, weight REAL, reps INTEGER
          )
        ''');
        await db.insert('sets', {'id': 'test-1', 'weight': 80.0, 'reps': 8});
      },
    );

    // Act: v2로 마이그레이션 (notes 컬럼 추가)
    await db.execute('ALTER TABLE sets ADD COLUMN notes TEXT');

    // Assert: 기존 데이터 보존 확인
    final rows = await db.query('sets');
    expect(rows.length, equals(1));
    expect(rows.first['weight'], equals(80.0));
    expect(rows.first['notes'], isNull); // 새 컬럼은 NULL
  });
}
```

---

## 3. 통합 테스트 — 세트 기록 → 1RM 계산 → 저장

> 파일: `integration_test/workout_flow_test.dart`

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:liftlog/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('should_show_1rm_after_recording_set', (tester) async {
    app.main();
    await tester.pumpAndSettle();

    // 벤치프레스 선택
    await tester.tap(find.text('벤치프레스'));
    await tester.pumpAndSettle();

    // 무게 입력
    await tester.enterText(find.byKey(Key('weight_input')), '80');
    await tester.enterText(find.byKey(Key('reps_input')), '8');

    // 기록 버튼
    await tester.tap(find.byKey(Key('record_btn')));
    await tester.pumpAndSettle();

    // 1RM 표시 확인
    expect(find.textContaining('93.3'), findsOneWidget);
  });
}
```

---

## 4. 테스트 커버리지 목표

| 레이어 | 목표 커버리지 | 현재 |
|--------|-------------|------|
| Domain (Epley 등) | 100% | - |
| Application (ViewModel) | 70%+ | - |
| Data (Repository) | 60%+ | - |
| Presentation (Widget) | 40%+ | - |

---

## 5. 좋은 테스트 작성 원칙

**이름 형식**: `should_[기대결과]_when_[조건]`

**AAA 패턴**:
```
// Arrange — 준비
// Act     — 실행
// Assert  — 검증
```

**외부 의존 격리**: 네트워크·DB는 Mock/Fake로 교체

---

## 6. CI 자동화 (선택)

> `.github/workflows/test.yml`

```yaml
name: Flutter Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
      - run: flutter pub get
      - run: flutter test
```

CI 뱃지를 README에 추가하면 발표 시 인상적입니다.
