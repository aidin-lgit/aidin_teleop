+++
title = "설치 및 설정"
weight = 1
+++

## 사전 준비

| 항목 | 버전 | 비고 |
|---|---|---|
| Python | 3.10 이상 | |
| `e2e_training` 패키지 | 최신 | 모델 정의·로봇 설정 공유 |
| ROS2 (`rclpy` + `sensor_msgs`) | Humble 이상 | 실로봇 연결 / 카메라 이미지 토픽 구독 |
| CUDA | 11.8 이상 | GPU 추론 권장 |

### 학습 결과물 확인

추론을 시작하려면 `e2e_training` 이 생성한 `run_dir` 이 다음 세 파일을 포함해야 합니다.
하나라도 빠지면 `e2e-infer` 실행이 실패합니다.

```
run_dir/
├── config.yaml     ← 학습 설정 (자동 생성)
├── stats.json      ← 정규화 통계 (자동 생성)
└── ckpt/
    └── best.pt     ← 체크포인트 (자동 생성)
```

## 설치

`e2e_inference` 는 `e2e_training` 의 모델 정의와 로봇 설정을 재사용합니다.
**같은 venv** 에 두 패키지를 설치하세요 (자세한 의존성 설치는 [학습 → 설치](../../training/environment/) 참고).

```bash
# 1) e2e_training 먼저 설치
cd <e2e_training_경로>
pip install -e ".[dev]"

# 2) e2e_inference 설치
cd <e2e_inference_경로>
pip install -e ".[dev]"

# 3) 설치 확인
e2e-infer --help
```

## Joint Name Map 설정

ROS2 `/joint_states` 의 joint 이름을, e2e_inference 가 사용하는 그룹 키
(`joint_arm_L`, `joint_arm_R`, `joint_hand_L`, `joint_hand_R`, `joint_head`, `joint_torso`)에
매핑하는 파일입니다. 그룹 키는 HDF5 obs 키 이름과 동일합니다. **로봇·체크포인트별로 1회** 작성합니다.

```bash
cd <e2e_inference_경로>
cp configs/inference/rby1_joint_names.example.yaml \
   configs/inference/rby1_joint_names.yaml
```

`rby1_joint_names.yaml` 을 열어 실제 로봇의 joint 이름을 채웁니다.

```yaml
joint_arm_L:
  - left_shoulder_pitch_joint
  - left_shoulder_roll_joint
  # ... (7개)
joint_arm_R:
  - right_shoulder_pitch_joint
  # ...
joint_hand_L:        # 15개 (엄지/검지/중지/약지/소지 각 3개)
  - left_thumb_joint1
  # ...
joint_hand_R:
  - right_thumb_joint1
  # ...
joint_head:  []      # 사용하지 않는 그룹은 빈 리스트
joint_torso: []
```

**작성 규칙**

- 그룹별 리스트의 **순서는 학습 시 데이터 수집 순서와 정확히 일치**해야 합니다
  (= 학습 `joint_keys`/`tcp_keys` 순서).
- 모든 그룹 joint 수의 총합은 `stats.json` 의 `obs.min` 배열 길이와 같아야 합니다.
- 사용하지 않는 그룹은 빈 리스트(`[]`)로 둡니다.

실제 발행 중인 joint 이름은 다음으로 확인합니다.

```bash
ros2 topic echo /joint_states --once
```

> 학습 데이터 / `rby1_joint_names.yaml` / 모델 가중치 차원이 일치하지 않으면 추론 시작 시
> 명확한 에러 메시지와 함께 즉시 종료되어, 잘못된 설정으로 로봇이 움직이는 것을 방지합니다.

### 다른 그리퍼/핸드로 교체 시

추론 코드는 손대지 않아도 됩니다. ① 새 하드웨어 데이터로 `e2e_training` 재학습 →
② 새 `stats.json` 이 자동으로 새 차원 정보 포함 → ③ `rby1_joint_names.yaml` 의
`joint_hand_L/R` 리스트만 실제 joint 이름으로 갱신.

## Runtime Config 설정 (publish 토글)

추론 동작(특히 **로봇 publish 여부**)을 코드 수정 없이 YAML 로 토글합니다.

```bash
cp configs/inference/runtime.example.yaml configs/inference/runtime.yaml
```

```yaml
# configs/inference/runtime.yaml
publish: false     # 안전 모드: 모델 예측만 콘솔 출력, 로봇 미동작
# publish: true    # 실제 구동: 액션을 로봇에 발행
```

| 값 | 동작 |
|---|---|
| `publish: false` | obs 수집 + 모델 추론은 그대로 수행. 액션은 콘솔에 그룹별 출력만, ROS2 토픽 발행 **안 함** |
| `publish: true` | 정상 동작. 액션을 `/aidin_rby1_joint_controller/joint_state_command` 로 발행 |

> `--runtime_config` 옵션을 생략하면 기본값은 `publish: true`(실제 발행)입니다.
> **새 체크포인트는 반드시 `publish: false` 로 먼저 출력값을 검증**하세요.

카메라 토픽도 `runtime.yaml` 에서 지정할 수 있습니다 (미지정 시 env 기본 토픽 사용).

```yaml
camera_topics:
  image_head_color:    /camera/head/color/image_raw
  image_wrist_L_color: /camera/left/color/image_raw
  image_wrist_R_color: /camera/right/color/image_raw
```
