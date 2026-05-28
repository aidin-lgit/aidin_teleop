+++
title = "Quick Start"
weight = 4
+++

> 이 문서는 시스템이 이미 설치·셋업되어 있다는 전제 하에 5분 안에 Teleop 을 한 번 돌려보기 위한 절차입니다.
> 처음 설치하는 경우 [하드웨어 셋업](../../hardware-setup/) → [소프트웨어 설치](../../software-install/) 를 먼저 진행하세요.

## 1. 하드웨어 ON

1. RBY1 UPC 전원 ON
2. RBY1 RPC 전원 ON
3. AIDIN Hand 전원 ON
4. VIVE 베이스 스테이션 전원 ON
5. VIVE Tracker 전원 ON
6. MANUS Glove 전원 ON
7. Meta Quest 부팅 후 브라우저 실행

## 2. Teleop PC 부팅

```bash
cd ~/Workspace/lgit_ws
source install/setup.bash
```

## 3. 하드웨어 인터페이스 실행

Terminal 1
```bash
cd ~/Workspace/lgit_ws/src/aidin_rby1_teleop_bringup
./robot_launch.sh
```

Terminal 2
```bash
ros2 launch aidin_rby1_teleop_bringup teleop_bringup.launch.py
```

Terminal 3
```bash
ros2 launch aidin_rby1_teleop_bringup teleop_control.launch.py vr_control:=true
```

> 자세한 실행 launch 파라미터 옵션은 [`aidin_rby1_teleop_bringup`](https://github.com/aidin-lgit/aidin_rby1_teleop_bringup) 패키지의 README 를 참고하세요.

## 4. 작업자 착용

- 양 손목 + 허리 VIVE Tracker 부착
- MANUS Glove 양손 착용
- Meta Quest HMD 착용

## 5. 원격 제어 및 데이터 로깅 시작

`aidin_rby1_vive_teleop` 노드가 다음과 같은 상태 머신으로 동작합니다.

```
        a                       c
 IDLE ─────► TELEOP ─────────────────────────► HOMING ─► IDLE
                                                  
   ▲ b: 어느 상태에서든 emergency_stop + 종료
```

키보드 단축키로 원격제어를 제어합니다. 한글 IME 상태(`ㅁ` / `ㅊ` / `ㅠ`)도 동일하게 인식됩니다.

| 동작 | 키 | 설명 |
| --- | --- | --- |
| 제어 및 로깅 시작 (`IDLE → TELEOP`) | `a` | 세션/녹화 ON, 목표 자세 발행 시작 |
| 중지 (`TELEOP → HOMING → IDLE`) | `c` | 발행/녹화/세션 OFF 후 홈 자세 복귀 |
| 긴급 정지 (모든 상태) | `b` | 세션 OFF + 로봇 긴급 정지 |

옵션으로 제공된 **발판 키보드(Foot Switch)** 를 사용할 경우 다음과 같이 매핑됩니다.

| 동작 | 페달 | 대응 키 |
| --- | --- | --- |
| 시작 | 왼쪽 페달 | `a` |
| 중지 | 오른쪽 페달 | `c` |
| 긴급 정지 | 가운데 페달 | `b` |

> 자세한 상태 전이, 키 매핑, ROS 2 서비스 호출 시점은 [`aidin_rby1_vive_teleop`](https://github.com/aidin-lgit/aidin_rby1_vive_teleop) README 의 "운영 (키보드 + 상태머신)" 절을 참고하세요.


## 6. 세션 → HDF5 변환

§5에서 `a` 키로 시작·`c` 키로 종료한 에피소드들은 `./logs/session_<timestamp>/episode_<N>/` 구조로 MCAP 파일로 저장됩니다. 세션이 끝난 뒤 해당 세션의 에피소드들을 RoboMimic 호환 HDF5 로 일괄 변환합니다.

```bash
# 가장 최신 session_* 자동 선택 → bundle 모드 (demo_0, demo_1, ... 가 누적된 단일 HDF5)
ros2 run ros2_mcap_recorder convert_session_to_hdf5

# 명시적 세션 지정 + per-episode 모드 (에피소드마다 별도 HDF5 파일)
ros2 run ros2_mcap_recorder convert_session_to_hdf5 \
  --session ./logs/session_20260524_153000 \
  --mode per-episode

# 샘플링 주파수를 30 Hz 로 override
ros2 run ros2_mcap_recorder convert_session_to_hdf5 --fps 30
```

출력 파일:

- `bundle` (기본): `<session>/dataset.hdf5` 안에 `data/demo_0`, `data/demo_1`, ... 그룹으로 누적
- `per-episode`: `<session>/dataset_episode_1.hdf5`, `dataset_episode_2.hdf5`, ...

> CLI 인자 전체 목록, 매핑 YAML 구성, LeRobot 변환 등 상세 사용법은 [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) README 를 참고하세요.

## 7. 학습 (e2e_training)

§6에서 만든 HDF5 데이터를 입력으로 Diffusion Policy 등 모방학습 모델을 학습합니다.

```bash
e2e-train -c configs/train/diffusion_policy.yaml \
  -o dataset.hdf5_paths=/path/to/dataset.hdf5
```

학습이 끝나면 `run_dir/` 아래 `config.yaml`, `stats.json`, `ckpt/best.pt` 가 생성됩니다 — 다음 단계 추론에 그대로 사용합니다.

> 학습 설정(YAML), 체크포인트 관리, GPU 메모리 튜닝, 새 모델/로봇 추가 등 상세 사용법은 [`e2e_training`](https://github.com/aidin-lgit/e2e_training) README 와 `docs/` 가이드를 참고하세요.

## 8. 추론 (e2e_inference)

학습된 체크포인트를 로드해 실로봇에서 추론·롤아웃을 실행합니다. 모델 예측 액션이 `/aidin_rby1_joint_controller/joint_state_command` 로 발행됩니다.

> ⚠️ 새 체크포인트는 반드시 `runtime.yaml` 의 **`publish: false`** (안전 모드) 로 먼저 콘솔 출력만 확인한 뒤 `publish: true` 로 전환하세요.

```bash
e2e-infer \
  --run_dir <e2e_training_run_dir> \
  --checkpoint best \
  --robot rb_y1 \
  --configs_dir ~/Workspace/lgit_ws/src/e2e_training/configs \
  --rby1_joint_name_map configs/inference/rby1_joint_names.yaml \
  --runtime_config configs/inference/runtime.yaml \
  --device cuda \
  --frequency 10 \
  --steps_per_inference 6 \
  --max_duration 60
```


> CLI 옵션 전체 목록, joint name map 작성법, 안전 검증 절차, 트러블슈팅은 [`e2e_inference`](https://github.com/aidin-lgit/e2e_inference) README 와 `docs/user_guide.md` 를 참고하세요.
