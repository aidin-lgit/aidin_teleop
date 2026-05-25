+++
title = "모델 배포"
weight = 1
+++

## 체크포인트 패키징

학습 종료 후 다음을 함께 묶어 배포 패키지 생성:

```
deploy/<task>_<version>/
├── checkpoint.ckpt
├── model.yaml            # 모델 설정 (학습 시 사용한 conf/model)
├── data_mapping.yaml     # 정규화/이미지 전처리 정보
├── action_postproc.yaml  # 액션 후처리 (스케일, EE 회전 표현 변환)
└── README.md
```

## 추론 노드 실행

```bash
ros2 launch aidin_rby1_teleop inference.launch.py \
    deploy_dir:=/data/deploy/pick_red_block_v3 \
    rate_hz:=30
```

노드 동작:

1. `deploy_dir` 의 yaml 들을 로드하고 모델 초기화
2. 학습 시 사용한 토픽과 **동일한 입력** 을 구독 (joint, image, head_pose 등)
3. 학습 시 사용한 토픽과 **동일한 출력** 을 발행 (`/teleop/cmd/...`)
4. 텔레옵 노드와 동일한 다운스트림 컨트롤러가 그대로 동작

즉, 학습-텔레옵-추론이 **같은 인터페이스**를 공유하므로 모드 전환 시 컨트롤러 측 변경이 없습니다.

## 모드 전환

- 텔레옵 → 추론: 텔레옵 disengage 후 inference engage
- 추론 → 텔레옵: 추론 disengage 후 텔레옵 engage
- 두 노드는 **동시에 engage 되지 않도록** 상호 배타 록(`/teleop/active_owner`) 사용
