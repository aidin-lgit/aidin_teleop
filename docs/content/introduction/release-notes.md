+++
title = "릴리즈 노트"
weight = 5
+++

2026-07-29 자 첫 정식 배포본 기준으로 정리한 변경 이력입니다. 각 패키지의 상세 변경 내역은 저장소의 `update.md` 를 참고하세요.

## 패키지 버전

`package.xml` 기준 버전입니다. `ros2 pkg xml <패키지명> --tag version` 으로 설치된 버전을 확인할 수 있습니다.

### 로봇측 (RBY1 UPC — `~/Workspace/aidin_ws`)


| 패키지 | 버전 | 역할 |
| --- | --- | --- |
| [`aidin_rby1_controller`](https://github.com/aidin-lgit/aidin_rby1_controller) | **1.1.1** | ros2_control 컨트롤러 (Cartesian IK / Admittance / Joint) |
| [`aidin_rby1_description`](https://github.com/aidin-lgit/aidin_rby1_description) | **1.1.0** | RBY1 + Aidin Hand URDF / xacro |
| [`aidin_rby1_hardware`](https://github.com/aidin-lgit/aidin_rby1_hardware) | 1.0.0 | RBY1 하드웨어 인터페이스 (real / virtual / isaac) |
| [`aidin_hand_controllers`](https://github.com/aidin-lgit/aidin_hand_controllers) | 1.0.0 | Aidin Hand 컨트롤러 (4계층, 1 kHz) |
| [`aidin_hand_description`](https://github.com/aidin-lgit/aidin_hand_description) | 1.0.0 | Aidin Hand URDF / xacro |
| [`aidin_hand_hardware`](https://github.com/aidin-lgit/aidin_hand_hardware) | 1.0.0 | Aidin Hand EtherCAT 하드웨어 인터페이스 |




### 원격제어측 (Teleop PC — `~/Workspace/lgit_ws`)

메타 저장소 [`aidin_rby1_teleop`](https://github.com/aidin-lgit/aidin_rby1_teleop) 아래 6개 서브모듈로 구성됩니다.

| 패키지 | 버전 | 역할 |
| --- | --- | --- |
| [`aidin_rby1_teleop_bringup`](https://github.com/aidin-lgit/aidin_rby1_teleop_bringup) | 1.0.0 | 통합 launch 진입점 |
| [`aidin_rby1_vive_teleop`](https://github.com/aidin-lgit/aidin_rby1_vive_teleop) | **1.1.0** | VIVE Tracker → 양팔·허리 제어 + 키보드 상태머신 |
| [`aidin_rby1_vr_teleop`](https://github.com/aidin-lgit/aidin_rby1_vr_teleop) | 1.0.0 | CloudXR VR 영상 송출 + 머리·양손 pose |
| [`vive_tracker_ros2`](https://github.com/aidin-lgit/vive_tracker_ros2) | 1.0.0 | OpenVR / SteamVR 트래커 드라이버 |
| `manus_ros2` ([`aidin_manus`](https://github.com/aidin-lgit/aidin_manus)) | 1.0.0 | MANUS Glove → 손가락 제어 |
| [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) | 1.0.0 | MCAP 녹화 + HDF5 / LeRobot 변환 |




### 학습·추론측 (Teleop PC — `~/Workspace/e2e_ws`)


| 패키지 | 버전 | 역할 |
| --- | --- | --- |
| [`e2e_training`](https://github.com/aidin-lgit/e2e_training) | 1.0.1 | 모방학습 학습 파이프라인 (`e2e-train`) |
| [`e2e_inference`](https://github.com/aidin-lgit/e2e_inference) | 1.1.0 | 학습 모델 추론 (`e2e-infer`) |


---



## 실행 방법에 영향을 주는 변경

기존 절차를 그대로 따르면 동작하지 않는 항목들입니다. 이전 버전 환경에서 올라오신 경우 이 절만 먼저 확인하세요.

### VR 스트리밍이 CloudXR 로 교체됨

Vuer 기반 경로(`teleop_stream.py`, `vr_teleop.py`, `vr_pose_publisher.py`, `vr_control.launch.py`)가 배포본에서 **제거**되고 NVIDIA CloudXR 스택으로 대체되었습니다.


| 구분       | 이전                                                    | 현재                                                                       |
| -------- | ----------------------------------------------------- | ------------------------------------------------------------------------ |
| 실행       | `ros2 run aidin_rby1_vr_teleop teleop_stream.py`      | `ros2 launch aidin_rby1_vr_teleop cloudxr_teleop.launch.py`              |
| HMD 접속   | `https://<IP>:8012` → vuer.ai → "VR 시작하기"             | `https://<IP>:48322` (인증서 수락) → `https://<IP>:8080` → CONNECT → Enter XR |
| 머리 자세 토픽 | `/hmd_pose`                                           | `/vr/hmd_pose`                                                           |
| 손 자세 토픽  | —                                                     | `/vr/{left,right}_hand_pose`, `/vr/{left,right}_hand_joints`             |
| 영상 토픽    | `/stereo/{left,right}/color/image_raw`                | `/zed/zed_node/stereo/color/rect/image` (`local`)                        |
| 화질 조정    | `vuer_image_scale` / `vuer_jpeg_quality` / `vuer_fps` | `image_transport`, `enable_video` + web client 설정                        |


CloudXR 런타임 SDK 와 TLS 인증서 발급이 선행 조건입니다. SDK 가 없는 PC 에서는 `use_vr_stream:=false` 로 제외하고 나머지 스택만 실행할 수 있습니다. 절차는 [원격제어 → 머리 카메라 제어 시 이미지 수신 방법](../../teleoperation/operation/#머리-카메라-제어-시-이미지-수신-방법) 참고.

### 컨트롤러 launch 인자 기본값

`bimanual_controller.launch.py` 를 인자 없이 실행하면 텔레옵 기본 설정과 맞지 않습니다.


| 인자                                 | 기본값     | 비고                                                                     |
| ---------------------------------- | ------- | ---------------------------------------------------------------------- |
| `use_wholebody_control`            | `True`  | 활성 Cartesian 컨트롤러가 `aidin_rby1_wholebody_controller` 가 되어 명령 토픽 이름이 바뀜 |
| `use_left_hand` / `use_right_hand` | `False` | 손가락 제어를 쓰려면 `True` 지정 필요                                               |
| `use_torso`                        | `True`  | —                                                                      |
| `hardware_mode`                    | `real`  | —                                                                      |


본 매뉴얼은 `use_wholebody_control:=False` (bimanual IK) 기준으로 작성되어 있습니다. wholebody IK 로 운용할 경우 명령 토픽·`safety_controllers`·녹화 토픽을 함께 맞춰야 합니다 ([상세](../../teleoperation/launch-sequence/#cartesian-컨트롤러-선택)).

### `virtual_mode` → `hardware_mode`

`aidin_rby1_controller` v1.1.0 / `aidin_rby1_description` v1.1.0 에서 launch·xacro 인자가 교체되었습니다. 구 인자를 넘기면 launch 가 에러로 알려줍니다.


| 이전                    | 현재                                           |
| --------------------- | -------------------------------------------- |
| `virtual_mode:=True`  | `hardware_mode:=virtual`                     |
| `virtual_mode:=False` | `hardware_mode:=real` (기본)                   |
| —                     | `hardware_mode:=isaac` (Isaac Sim 토픽 연동, 신규) |




### 인터페이스 launch 구성 — 카메라 분리

`teleop_interface.launch.py` / `teleop_bringup.launch.py` 에 **RealSense 카메라가 포함되지 않습니다.** 카메라는 별도 터미널에서 `rs_multi.launch.py` 로 실행하세요.

```bash
ros2 launch aidin_rby1_teleop_bringup rs_multi.launch.py
```



### 키보드 상태머신 — 에피소드 반복은 `a`

`aidin_rby1_vive_teleop` 의 상태 전이는 `IDLE → SESSION → TELEOP → HOMING` 4단계입니다.


| 키   | TELEOP 상태에서의 동작                              |
| --- | -------------------------------------------- |
| `a` | 녹화 종료 + 호밍, **세션 유지** → 같은 세션에 다음 에피소드 이어 녹화 |
| `c` | 녹화 종료 + **세션 닫기** + 호밍                       |


에피소드 사이에 `c` 를 누르면 세션이 닫혀 다음 `a` 가 새 세션 폴더를 만들고, 세션 일괄 변환에서 한 번에 묶이지 않습니다.

### MCAP 출력 경로

실행 방식에 따라 기본 경로가 다릅니다.


| 실행 방식                                                           | 기본 출력 경로       |
| --------------------------------------------------------------- | -------------- |
| `ros2 launch ... logger.launch.py` / `teleop_bringup.launch.py` | `~/data/mcap/` |
| `ros2 run ros2_mcap_recorder mcap_recorder`                     | `./logs/`      |


변환 도구는 `./logs` 를 기준으로 세션을 자동 탐색하므로, launch 로 녹화한 세션은 `--logs-dir ~/data/mcap` 또는 `--session` 을 명시해야 합니다 ([상세](../../data-collection/postprocessing/)).

### 통신 미들웨어 — Zenoh

DDS 기본 구현체가 Zenoh 로 통일되었습니다. 로봇 PC 와 Teleop PC 양쪽에서 `RMW_IMPLEMENTATION=rmw_zenoh_cpp` 와 동일한 `ROS_DOMAIN_ID` 를 사용해야 토픽이 보입니다. `robot_launch.sh` 는 로컬·원격에 같은 값을 적용합니다. 설정은 [통신 설정](../../software-install/configuration/) 참고.

---



## 패키지별 주요 변경



### `aidin_rby1_controller` 1.1.1

- **안전 정지 상태 토픽** `~/safety_status` (`diagnostic_msgs/DiagnosticStatus`, latched) 신규 — `level` 로 판별 (0 = 정상, 1 = 입력 끊김, 2 = 안전 정지 latch). 정지 원인·limb·측정값 동봉, 기본 10 Hz 발행
- `hardware_mode` 로 백엔드 선택 (`real` / `virtual` / `isaac`), Isaac 연동 인자 3종 추가
- 허리 입력 축 마스크 `ik_config.torso.input_mask` — 기본은 높낮이 + 좌우 회전만 사용
- 팔 타깃 기준 프레임 선택 `ik_config.arms_relative_to_torso`
- 몸통 기울기 제한 `combined.body_tilt_limit` (wholebody 전용, 기본 앞뒤 ±20° / 좌우 ±10°)
- `use_self_collision` launch 인자로 자기 충돌 회피 on/off



### `aidin_rby1_description` 1.1.0

- `hardware_mode` 하나로 본체·손 `<ros2_control>` 플러그인 결정
- Isaac Sim 브리지 플러그인 + USD 모델(`model/usd/`)
- 플랫폼 선택 `model_type` (`rby1a` 차동 2륜 / `rby1m` 메카넘 4륜), 손 선택 `hand_model` (`aidin_gen1` / `tesollo_dg5f`)
- 머리 상태 및 `ee_left` / `ee_right` EEF 프레임 추가 — 컨트롤러 pose broadcaster 기준
- 손목 카메라 지그, 커넥터/링 메시 추가
- rby1a 본체를 벤더 model_v1_2 기준으로 정렬 (어깨 프레임, `arm_6` 관절 한계 ±155°, 팔 관성값)



### `aidin_rby1_hardware` 1.0.0

- `SystemInterface` 플러그인 3종 — 실기(rby1-sdk) / 가상 / Isaac 브리지
-  `~/robot_status`, `~/wifi_status` JSON 토픽, 전원·서보·WiFi 명령 서비스. 블로킹 SDK 호출은 전용 executor 스레드에서 처리되어 제어 루프를 막지 않음
- `~/home_to_initial` homing 서비스, `~/emergency_stop` 비상 정지 서비스



### `aidin_hand_controllers` 1.0.0

- EtherCAT 하드웨어부터 joint 명령까지 4계층 구성 (`ActuatorController` → `HandKinematicsController` → `JointCommandController`, 1000 Hz)
- 양손 joint 명령 인터페이스 `/hand_joint_controller/joint_state_command`
- 손별 homing (`/<side>_actuator_controller/homming`) · 캘리브레이션 (`/<side>_hand_kinematics_controller/calibration`) 서비스. 캘리브레이션 값은 `config/hand_calibration.yaml` 에 저장
- 손 F/T 센서 broadcaster, 엔코더 상태 broadcaster



### `aidin_hand_hardware` 1.0.0

- `SystemInterface` 플러그인 3종 — EtherLab / ethercat_driver_ros2 / 가상
- 기동 시 homing 시퀀스, 완료를 latched 토픽으로 발행



### `aidin_hand_description` 1.0.0

- Aidin Hand 양손 URDF/xacro (5지 × 3관절), 손끝·촉각 센서 프레임 제공



### `aidin_rby1_teleop_bringup` 1.0.0

- `teleop_bringup.launch.py` — 글러브·트래커·CloudXR·MCAP 로깅 일괄 실행 (`use_vr_stream`, `endpoint_ip`, `config`, `output_dir`, `image_format`)
- `teleop_control.launch.py` — 손가락·양팔·허리·머리 제어 일괄 실행, 하위 launch 인자 위임
- `robot_launch.sh` — SSH 로 로봇 PC 컨트롤러 실행. `Ctrl+C` 한 번으로 종료, `REMOTE_HOST` / `REMOTE_USER` / `REMOTE_PASS` 환경변수 지원
- 로컬과 원격이 같은 `ROS_DOMAIN_ID` · `RMW_IMPLEMENTATION` 을 쓰도록 통일 — 이전에는 원격 도메인이 하드코딩되어 토픽이 보이지 않을 수 있었음
- `rs_multi.launch.py` — RealSense head/left/right 를 `use_<이름>` / `<이름>_serial` 로 선택 실행



### `aidin_rby1_vive_teleop` 1.1.0

- **컨트롤러 안전 정지 연동** — `<컨트롤러>/safety_status` 를 구독해 TELEOP 중 `a` 를 누르면 정지 상태일 때 에피소드를 끝내지 않고 `resume` 호출. 정지가 아니면 기존대로 녹화 종료 + 호밍
- `keyboard_mode` (`global` / `terminal` / `off`), `auto_start`, `control_torso` 인자



### `aidin_rby1_vr_teleop` 1.0.0

- CloudXR 통합 런치 `cloudxr_teleop.launch.py` — 런타임 · WSS 프록시 · web client · OpenXR 송출 앱 4개 프로세스 일괄 기동
- `camera_location` 으로 ZED 연결 위치 선택 (`local` / `remote`), `image_transport` 로 `compressed` / `raw` 선택
- 헤드셋 브라우저용 CloudXR.js WebXR 클라이언트 — 앱 설치 없이 URL 접속



### `vive_tracker_ros2` 1.0.0

- `vive_tracker_node` — raw / calibrated 자세와 twist 를 **100 Hz** 로 발행, TF broadcast
- 세션 단위 CSV 로깅 — `/vive_log_controller/set_csv_logging` (SetBool) 로 모든 트래커 동시 제어

### `manus_ros2` (aidin_manus) 1.0.0

- `manus_data_publisher` — 글러브 데이터를 `/manus_glove_<N>` 으로 발행, `/manus_glove_<N>/vibration_cmd` 로 진동 명령 수신. 토픽 번호는 연결 순서이며 좌/우는 메시지의 `side` 필드로 판별

### `ros2_mcap_recorder` 1.0.0

- `mcap_recorder` — 서비스 기반 MCAP 레코더. `~/session_control` / `~/record_control` 로 `session_<날짜>/episode_<N>/` 자동 생성. `topics.yaml` 의 토픽이 모두 잡힐 때까지 1초마다 재시도하며 TRANSIENT_LOCAL 토픽도 QoS 를 맞춰 구독
- `--image-format` 으로 컬러 이미지를 `compressed`(기본) / `raw` 중 선택 — yaml 수정 없이 전환

---

