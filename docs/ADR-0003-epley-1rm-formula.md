# ADR-0003: Epley 공식을 1RM 추정에 사용

- 상태: 채택됨 (Accepted)
- 날짜: 2026-03-18
- 결정자: whm1218-png
- 관련 문서: docs/architecture.md, lib/domain/services/one_rep_max_service.dart

## 컨텍스트 (배경)

LiftLog는 사용자가 입력한 무게와 반복 횟수로부터 **1RM(1 Repetition Maximum, 1회 들 수 있는 최대 무게)** 을 자동 계산해야 한다.

이유:
1. 1RM은 헬스에서 가장 보편적인 강도 지표다 (다음 세션 무게 결정의 기준).
2. 실제 1RM을 매번 측정하는 것은 부상 위험이 크다. 따라서 8~12회 반복 가능한 무게로부터 **추정**하는 공식이 필요하다.

## 검토한 대안

1RM 추정 공식은 여러 가지가 있다. 각각의 특징:

| 공식 | 수식 | 정확도(8~12회) | 단순함 |
|------|-----|---------------|--------|
| **Epley** | `weight × (1 + reps/30)` | 매우 정확 | 매우 단순 |
| Brzycki | `weight × 36/(37 - reps)` | 정확 | 보통 |
| Lombardi | `weight × reps^0.10` | 보통 | 복잡 (지수 연산) |
| O'Conner | `weight × (1 + reps/40)` | 정확 | 단순 |
| Mayhew | `100 × weight / (52.2 + 41.9 × e^(-0.055 × reps))` | 정확 | 매우 복잡 |

비교 기준:
- **정확도**: 일반인이 가장 많이 시도하는 8~12회 범위에서 실측 1RM과의 오차
- **단순함**: 사용자에게 "어떻게 계산되었는가?"를 설명할 때 직관성

## 결정

**Epley 공식 `1RM = weight × (1 + reps/30)`을 사용한다.**

이유:
1. **계산이 단순하고 직관적이다.** "30회 중 몇 회 했는가"로 해석 가능해 사용자에게 설명하기 쉽다.
2. **8~12회 범위에서 다른 공식과 거의 동일한 정확도**를 보인다.
3. **헬스 커뮤니티에서 가장 널리 인용되는 공식**으로, 사용자에게 친숙하다.
4. **reps == 1 인 경우 자연스럽게 weight를 반환**한다 (실측 1RM이므로 추정 불필요).

## 결과

- `OneRepMaxService.estimate(weight, reps)` 메소드 구현
- reps == 1 인 경우 weight 그대로 반환
- reps > 12 인 경우 "추정값 오차가 클 수 있음" 경고 표시 (`isHighRepWarning`)
- reps == 0 또는 weight ≤ 0 인 경우 `ArgumentError` 발생
- 단위 테스트: `test/domain/one_rep_max_service_test.dart` (9개 케이스)

## 향후 검토

- v2에서 사용자 선택 기능 추가 고려 (Epley / Brzycki 토글)
- 사용자가 실측 1RM을 입력했을 때 그 값을 기준으로 보정 가능

## 참고

- Epley, B. (1985). Poundage Chart. Boyd Epley Workout. Lincoln, NE.
- 위키피디아 "One-repetition maximum"
