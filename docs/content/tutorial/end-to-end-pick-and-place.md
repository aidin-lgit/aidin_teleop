+++
title = "End-to-End: Pick & Place"
weight = 1
+++

> 본 튜토리얼은 **테이블 위의 과일 2개를 상자 안에 차례로 넣는 작업**을 통해 원격제어 → 데이터 수집 → MCAP→HDF5 변환 → 학습 → 자율 추론까지 전체 파이프라인을 한 번 돌려보는 것이 목적입니다.

## 0. 준비물

### 시스템

- RBY1 + AIDIN Hand (양팔제어 모드), VIVE × 3, MANUS Glove
- Teleop PC 워크스페이스 빌드 완료 ([소프트웨어 설치 → 워크스페이스 구성 및 빌드](../../software-install/workspace/) 참고)

### 작업 환경

- **상자 1개** — 로봇 정면 작업 영역 안에 배치
- **과일 2개** 

## 1. 하드웨어 ON

1. RBY1 RPC 전원 ON
2. RBY1 UPC 전원 ON
3. AIDIN Hand 전원 ON
4. VIVE 베이스 스테이션 전원 ON
5. VIVE Tracker 전원 ON (양 손목 + 허리)
6. MANUS Glove 전원 ON

## 2. Teleop PC 부팅

```bash
cd ~/Workspace/lgit_ws
source install/setup.bash
```

## 3. 하드웨어 인터페이스 실행

Terminal 1 — 로봇 제어기

```bash
cd ~/Workspace/lgit_ws/src/aidin_rby1_teleop_bringup
./robot_launch.sh
```

Terminal 2 — 입력 디바이스 brings-up

```bash
ros2 launch aidin_rby1_teleop_bringup teleop_bringup.launch.py
```

Terminal 3 — 원격제어 노드

```bash
ros2 launch aidin_rby1_teleop_bringup teleop_control.launch.py
```

> 자세한 launch 인자는 [`aidin_rby1_teleop_bringup`](https://github.com/aidin-lgit/aidin_rby1_teleop_bringup) README 를 참고하세요.

## 4. 작업자 착용

- 양 손목 + 허리 VIVE Tracker 부착 (허리 트래커 LED 가 위로 향하도록)
- MANUS Glove 양손 착용
- Meta Quest HMD 착용 후 [Meta Quest 셋업 → 시점 영상 수신](../../hardware-setup/meta-quest/#시점-영상-수신) 절차로 스트리밍 연결

## 5. 원격제어로 작업 시연 + 데이터 수집

`aidin_rby1_vive_teleop` 노드의 키보드 상태 머신을 사용해 세션·에피소드를 제어합니다. 한글 IME 상태에서도 `ㅁ` / `ㅊ` / `ㅠ` 로 동일하게 동작합니다.

| 동작 | 키 / 페달 |
| --- | --- |
| 세션 시작 + 에피소드 녹화 시작 + 제어 시작 | `a` (왼쪽 페달) |
| 에피소드 종료 → 홈 복귀 | `c` (오른쪽 페달) |
| 긴급 정지 + 종료 | `b` (가운데 페달) |

### 5-1. 한 에피소드 시연 흐름

1. **시작 자세 정렬** — 작업자는 로봇 초기 자세에 맞춰 양손을 가슴 앞에 모으고 정면(머리 제어 모드일 때)을 바라봄. 로봇과 같은 방향을 볼 필요는 없음.
2. **`a` 입력** — 세션이 열리고 첫 에피소드 녹화 + 제어가 시작됨. 이 시점의 허리·머리 트래커 자세가 reference frame 으로 캡처됨.
3. **작업 수행** — 과일 1을 집어 상자에 넣고, 이어서 과일 2를 집어 상자에 넣은 뒤 손을 중립 위치로 복귀.
4. **`c` 입력** — 에피소드 녹화가 닫히고 로봇이 홈으로 복귀.
5. **상자/과일 재배치** — 과일 위치를 변경하고, 다시 `a` 를 눌러 다음 에피소드 녹화 시작.

세션을 완전히 끝낼 때는 마지막으로 `c` 를 한 번 더 누릅니다. 전체 흐름과 상태 머신 상세는 [원격제어 → 원격제어 및 녹화 작업 프로세스](../../teleoperation/operation/#원격제어-및-녹화-작업-프로세스) 참고.

### 5-2. 권장 수집 분량

- **최소 30 에피소드** 부터 시작, 학습 결과를 보고 추가 수집 권장
- 매 에피소드마다 **과일 위치·자세 변경**, 작업자 시점 약간 변경 → 분포 다양성 확보
- 성공한 에피소드만 보존 (실패는 다음 변환 단계에서 폴더 통째로 제거)

수집된 데이터는 `./logs/session_<timestamp>/episode_<N>/episode_<N>_0.mcap` 구조로 자동 저장됩니다.

## 6. 세션 → HDF5 변환

세션이 끝난 뒤 한 번의 명령으로 모든 에피소드를 RoboMimic 호환 HDF5 로 변환합니다.

```bash
# 가장 최신 session_* 자동 선택 → bundle (단일 HDF5 에 demo_0, demo_1, ... 누적)
ros2 run ros2_mcap_recorder convert_session_to_hdf5

# 명시적 세션 지정
ros2 run ros2_mcap_recorder convert_session_to_hdf5 \
  --session ./logs/session_20260524_153000
```

출력은 `<session>/dataset.hdf5` 한 파일에 모든 에피소드가 `data/demo_<i>/` 그룹으로 누적됩니다. 변환 후에는 [myHDF5](https://myhdf5.hdfgroup.org/) 에 업로드하여 shape·attribute·시간축 길이 등을 점검하세요. 자세한 사용법은 [데이터 수집 → 후처리 및 검증](../../data-collection/postprocessing/) 참고.

## 7. Diffusion Policy 학습

학습용 PC 의 가상환경을 활성화한 뒤 `e2e_training` 의 `e2e-train` 으로 학습을 시작합니다.

```bash
cd ~/Workspace/e2e_ws
e2e-train \
  -c configs/train/diffusion_policy.yaml \
  -o dataset.hdf5_paths=<HDF5_PATH> \
  -o epochs=3 \
  -o batch_size=4
```

- 학습이 끝나면 `e2e_training/artifacts/train/diffusion_policy` 경로에 `ckpt/best.pt` 와 `ckpt/last.pt` 가 생성됩니다.
- 학습 epoch 수, GPU 메모리 튜닝, EMA, resume 등 옵션은 [`e2e_training`](https://github.com/aidin-lgit/e2e_training) README 와 `docs/` 가이드를 참고하세요.

## 8. 자율 추론 (실로봇)

학습된 체크포인트를 로드해 동일한 ROS 2 인터페이스로 자율 Pick & Place 를 수행합니다.

> ⚠️ 새 체크포인트는 반드시 `runtime.yaml` 의 **`publish: false`** (안전 모드) 로 먼저 콘솔 출력만 확인한 뒤 `publish: true` 로 전환하세요.

```bash
cd ~/Workspace/e2e_ws
e2e-infer \
  --run_dir /home/cobot-ai/Workspace/e2e_ws/e2e_training/artifacts/train/diffusion_policy/<CHECKPOINT_DIR> \
  --checkpoint last \
  --robot rb_y1 \
  --configs_dir /home/cobot-ai/Workspace/e2e_ws/e2e_training/configs \
  --rby1_joint_name_map configs/inference/rby1_joint_names.yaml \
  --runtime_config configs/inference/runtime.yaml \
  --device cuda
```

`publish: false` 동안 콘솔에 그룹별 액션 값이 찍히므로 관절 한계·NaN·점프 여부를 확인합니다. 안전이 확보되면 `publish: true` 로 전환하고 짧은 `--max_duration` (예: 10초) 부터 검증한 뒤 본 작업을 진행하세요. CLI 옵션·트러블슈팅 상세는 [`e2e_inference`](https://github.com/aidin-lgit/e2e_inference) README 와 `docs/user_guide.md` 참고.


