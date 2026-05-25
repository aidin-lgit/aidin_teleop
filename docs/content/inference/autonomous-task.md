+++
title = "자율 작업 수행"
weight = 2
+++

## 사전 점검

- 작업 환경(물체 위치, 배경, 조명)이 학습 시 분포 범위 내에 있는가
- 카메라 시점이 학습 시점과 일치하는가 (Quest 카메라 캘리브레이션·자세)
- 정규화 통계가 학습 시 사용한 것과 같은가 (`data_mapping.yaml` 점검)

## 실행 절차

1. 추론 노드 실행 (참고: [모델 배포](../model-deploy/))
2. RViz 또는 GUI 에서 모델 출력 액션을 사전 모니터 (E-Stop 손가락 위)
3. 텔레옵 → 추론 모드로 전환 (engage)
4. 작업 수행 관찰, 필요 시 즉시 disengage 후 텔레옵 인계

## 한 사이클 = 한 에피소드

- 한 번의 실행은 학습 시 한 에피소드 길이와 유사하도록 운영
- 종료 후 자동 disengage 또는 사용자 disengage

## 분포 외(OOD) 동작 감지

- 추론 노드는 action norm / obs norm 이 학습 분포의 상위 백분위를 넘어서면 경고
- 임계 초과 시 자동 disengage 옵션 활성화 가능 (`safety.auto_disengage.enabled=true`)

> *TODO: 임계값 권장치, 경고 토픽/UI 표시 방식*
