# AURIX TC387 ACC 개발대회

> 3학년 현대제어 수업 · Infineon ACC 개발대회 · 3인 팀 · **14개 팀 중 4등**
>
> 담당: Simulink C 코드 생성, AURIX TC387 보드 실장, 성능 검증

---

## 프로젝트 개요

MATLAB Simulink로 설계한 ACC(Adaptive Cruise Control) 제어 로직을 C 코드로 생성해 Infineon AURIX TC387 보드에 실장하고,
MIL → PIL 단계로 검증(Prescan 시뮬레이션)한 프로젝트입니다.

- **Tools**: MATLAB / Simulink, Prescan
- **Target**: Infineon AURIX TC387 (TriCore)

---

## 1. MIL–PIL 불일치 해결

MIL에서는 정상이던 로직이 PIL(Prescan 시뮬레이션)에서는 의도한 시점에 동작하지 않는 지연 문제가 발생했습니다.

| 단계 | 한 일 | 확인한 것 |
|---|---|---|
| MIL | Matlab Scope로 데이터를 구간별로 관측 | 제어 로직 자체에는 이상 없음 |
| PIL | STM 타이머로 수신·송신 실행 시간 측정 | 지연이 타깃 HW 실행 구간에서 발생 |
| 원인 분석 | 메모리 정렬 가설 → User Manual 확인 후 배제. 이후 Application Notes를 참고해 지연 요소를 하나씩 소거 | **TC387 FPU는 단정밀도만 지원** → `double` 연산이 SW 에뮬레이션으로 처리되며 오버헤드 발생 |
| 해결 | 변수 자료형을 `double` → `float`로 변경 | 정상 동작 확보 |

**결과**: ACC 정적 상태 오차 **13% 미만** 달성

---

## 2. 맵 데이터 기반 NSCC 제안

- **문제**: 코너 구간에서 거리 센서값이 유실되어 목표 거리가 벌어졌습니다.
- **해결**: 3주간 Simulink의 센서 데이터를 하나씩 확인해 맵 데이터를 활용할 수 있다는 점을 찾았고, 현업에서 **NSCC**(Navigation-based Smart Cruise Control)라는 이름으로 쓰이는 방식임을 확인해 제안했습니다.
- **결과**: NSCC 로직의 독창성을 인정받아 **14개 팀 중 4등**을 기록했습니다.

---

> 대회 제출 코드는 정리한 뒤 추가할 예정입니다.
