+++
title = "안전 모니터링 / 성능 평가"
weight = 3
+++

## 런타임 안전 모니터링

- **속도/가속도 클립**: 컨트롤러 측에서 EE 속도·관절 가속도 상한 강제
- **워크스페이스 한계**: EE 목표가 사전 정의된 박스(box) 밖이면 차단
- **그리퍼 힘 한계**: AIDIN Hand 의 토크 상한
- **OOD 경보**: [자율 작업 수행 → OOD 감지](../autonomous-task/#분포-외ood-동작-감지) 참고

## 성능 평가 프로토콜

### 1. 정량 지표 (오프라인)

- `val_mse_action` (테스트 split 액션 MSE)
- 액션 분포의 KL divergence (학습 vs 모델 예측)

### 2. 정량 지표 (실로봇)

- 성공률: N 회 시도 중 성공 비율 (성공 정의는 작업별 별도 명시)
- 평균 사이클 시간
- 인간 개입 발생 횟수

### 3. 평가 시트 (권장)

```csv
exp_id,task,n_trials,success,avg_cycle_s,interventions
dp_v3,pick_red_block,30,27,11.4,2
act_v2,pick_red_block,30,24,12.1,5
```

## 정기 회귀 평가

- 데이터셋 또는 모델 코드 변경 시 동일 평가 세트로 회귀 확인
- 평가 세트는 `tests/eval_set/<task>.yaml` 에 정의 (물체 위치/방향/조명 조합)
