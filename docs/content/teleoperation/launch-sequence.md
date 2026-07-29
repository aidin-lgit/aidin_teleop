+++
title = "부팅 시퀀스"
weight = 1
+++

## 권장 순서

1. **로봇 플랫폼**
   - RBY1 RPC 전원 ON
   - RBY1 UPC 전원 ON
   - AIDIN Hand 전원 ON
   - 로봇 제어기 실행 — Teleop PC 에서 `robot_launch.sh` 를 쓰면 SSH 접속과 도메인·미들웨어 통일이 함께 처리됩니다.
   ```bash
   cd ~/Workspace/lgit_ws/src/aidin_rby1_teleop/aidin_rby1_teleop_bringup
   ./robot_launch.sh use_left_hand:=True use_right_hand:=True use_wholebody_control:=False
   ```

   `Ctrl+C` 한 번으로 원격 컨트롤러가 정상 종료됩니다.

   #### 주요 launch 파라미터

   `robot_launch.sh` 는 전달받은 `<이름>:=<값>` 인자를 원격의 `bimanual_controller.launch.py` 로 그대로 위임합니다. 운영 중 자주 쓰는 인자는 다음과 같습니다.

   | 인자 | 기본값 | 설명 |
   | --- | --- | --- |
   | `hardware_mode` | `real` | 하드웨어 백엔드. `real`(실기) / `virtual`(명령을 상태로 되먹임, RViz + rqt 동반) / `isaac`(Isaac Sim 토픽 연동). `real` 이 아니면 손도 가상으로 동작 |
   | `use_left_hand` / `use_right_hand` | `False` | 손 포함 여부. **MANUS 손가락 제어를 쓰려면 `True` 로 지정** |
   | `use_torso` | `True` | 허리 사용 여부. `False` 면 허리는 고정되고 팔만 추종 |
   | `use_left_arm` / `use_right_arm` | `True` | 팔 서브체인 활성화 |
   | `use_head` | `True` | 머리 서브체인 활성화. `vr_control` 로 머리를 제어할 때 필요 |
   | `use_mobile` | `True` | 모바일 베이스 컨트롤러 spawn |
   | `use_wholebody_control` | `True` | 허리 + 양팔 통합 IK 사용. **활성 컨트롤러 이름이 바뀌므로 아래 표 확인** |
   | `use_admittance_control` | `False` | 손목 F/T 기반 admittance 레이어 사용 |
   | `use_self_collision` | `True` | 자기 충돌 회피 (bimanual / admittance 컨트롤러 전용) |
   | `robot_ip` | `192.168.30.1:50051` | 실기 모드에서 RBY1 본체 통신 endpoint |
   | `hand_model` | `aidin_gen1` | 손 프로파일. `aidin_gen1` / `tesollo_dg5f` |
   | `model_type` | `rby1a` | 플랫폼. `rby1a`(차동 2륜) / `rby1m`(메카넘 4륜) |
   | `initial_positions_file` | `initial_positions.yaml` | Homing 목표가 되는 초기 자세 yaml |
   | `control_frequency` | `500` | 하드웨어 제어 루프 주기 [Hz] |
   | `arm_stiffness` / `torso_stiffness` | `400.0` / `100.0` | URDF 로 전달되어 하드웨어 게인에 반영 |
   | `damping_ratio` | `0.1` | 동일 |
   | `controller_manager_spawn_delay` | `2.0` | 컨트롤러 spawner 호출 지연 [s]. spawner timeout 이 나면 값을 올릴 것 |


   #### Cartesian 컨트롤러 선택

   두 인자의 조합으로 활성 Cartesian 컨트롤러가 결정되고, 텔레옵이 명령을 보낼 토픽 이름(`<컨트롤러>/pose_array_command`)도 함께 바뀝니다.

   | `use_wholebody_control` | `use_admittance_control` | 활성 컨트롤러 |
   | --- | --- | --- |
   | `False` | `False` | `aidin_rby1_bimanual_controller` ← **본 매뉴얼 기준** |
   | `False` | `True` | `aidin_rby1_cartesian_admittance_controller` |
   | `True` | `False` | `aidin_rby1_wholebody_controller` (launch 기본값) |
   | `True` | `True` | `aidin_rby1_wholebody_cartesian_admittance_controller` |

   본 매뉴얼은 `use_wholebody_control:=False` 를 기준으로 작성되어 있습니다 — 텔레옵의 `pose_command_topic` 기본값과 녹화 토픽 화이트리스트가 이 구성에 맞춰져 있습니다.

   Homing 서비스는 컨트롤러 종류와 무관하게 `/aidin_rby1_bimanual_controller/home_to_initial` 로 고정 노출됩니다.

   {{% notice style="warning" title="wholebody IK 로 운용할 때 함께 맞춰야 할 것" %}}
   `use_wholebody_control` 을 기본값(`True`) 으로 두면 활성 컨트롤러가 `aidin_rby1_wholebody_controller` 가 되므로, 텔레옵 측에서 세 가지를 함께 지정해야 합니다.

   - **명령 토픽** — `teleop_control.launch.py` 의 `pose_command_topic` 기본값이 bimanual 이므로 `pose_command_topic:=/aidin_rby1_wholebody_controller/pose_array_command` 로 바꿔야 로봇이 움직입니다.
   - **안전 정지 resume** — `aidin_rby1_vive_teleop` 의 `safety_controllers` 기본 목록에 `/aidin_rby1_wholebody_controller` 가 없어, 그대로 두면 안전 정지 시 `a` 로 resume 이 되지 않습니다.
   - **녹화 토픽** — `ros2_mcap_recorder` 의 `topics.yaml` 이 bimanual 명령 토픽을 기록하므로, 수정 후 재빌드하지 않으면 액션 데이터가 비어 있게 됩니다.
   {{% /notice %}}

   #### 스크립트 자체 옵션 · 환경변수

   접속 대상과 통신 설정은 환경변수로 덮어씁니다.

   | 환경변수 | 기본값 | 설명 |
   | --- | --- | --- |
   | `REMOTE_HOST` | `192.168.2.21` | 로봇 PC (UPC) 주소 |
   | `REMOTE_USER` | `nvidia` | 로봇 PC 계정 |
   | `REMOTE_PASS` | 스크립트 내 값 | 로봇 PC 비밀번호 |
   | `ROS_DOMAIN_ID` | `1` | 로컬·원격이 **함께** 쓰는 도메인 |
   | `RMW_IMPLEMENTATION` | `rmw_zenoh_cpp` | 미들웨어 (로컬·원격 동일) |

   스크립트 플래그는 `--no-camera` (로컬 카메라 실행 생략) 와 `-h` (도움말) 두 개입니다.

   ```bash
   # 도메인을 11 로 바꿔 실행 (로컬·원격 모두 적용)
   ROS_DOMAIN_ID=11 ./robot_launch.sh use_left_hand:=True use_right_hand:=True

   # 실기 없이 가상 모드로 동작 확인 (RViz + rqt_controller_manager 동반)
   ./robot_launch.sh hardware_mode:=virtual use_wholebody_control:=False

   # 다른 로봇 PC 로 접속
   REMOTE_HOST=192.168.2.30 ./robot_launch.sh use_left_hand:=True use_right_hand:=True

   ./robot_launch.sh -h        # 도움말
   ```

   SSH 접속에 `sshpass` 를 사용하므로 미설치 시 `sudo apt install sshpass` 를 먼저 실행하세요. 게인·admittance·IK 튜닝 파라미터를 포함한 전체 인자 목록은 [`aidin_rby1_controller`](https://github.com/aidin-lgit/aidin_rby1_controller) README 를, UPC 에 직접 접속해 실행하는 방법은 [Quick Start §1](../../introduction/quick-start/#1-로봇-컨트롤러-rby1-upc) 을 참고합니다.

   {{% notice style="warning" title="`model_version` 기본값 확인 필요" %}}
   `model_version` 기본값은 `1.2` 이나, launch 파일 주석에 "`rby1a/model_v1_0` 만 `FT_sensor_R` 를 가지며 `1.2` 에서는 non-virtual 하드웨어의 `MakeState` 가 실패한다" 는 경고가 있습니다. 실기 기동이 하드웨어 초기화 단계에서 실패하면 `model_version:=1.0` 을 시도해 보세요.
   {{% /notice %}}

   {{% notice style="warning" title="Homing 자세 이동 시 안전 확인" %}}
   위 launch 실행 시 로봇이 [`initial_positions.yaml`](https://github.com/aidin-lgit/aidin_rby1_description/blob/main/model/urdf/initial_positions.yaml) 에 정의된 초기화 자세로 즉시 Homing 합니다. 양 팔·허리·헤드가 모두 동시에 움직이므로 **실행 전 로봇 주변과 이동 경로상에 사람·장애물이 없는지 반드시 확인**하고, E-Stop 이 손에 닿는 위치에 있는지 점검하세요.
   {{% /notice %}}

2. **트래킹 장비**
   - VIVE Lighthouse 베이스 ON
   - SteamVR 실행 후 3개 트래커 (좌 손목 / 우 손목 / 허리) 모두 녹색 확인

     ![SteamVR 트래커 상태](/images/streamvr.png?width=300px)

   - MANUS Glove ON, Core 에서 페어링 확인
   - Meta Quest 부팅 후 브라우저 앱 실행 (Ego-view 원격제어 모드 시)
3. **원격제어 PC — 카메라**

   RealSense 3대는 인터페이스 launch 에 포함되지 않으므로 별도 터미널에서 실행합니다.

   ```bash
   cd ~/Workspace/lgit_ws && source install/setup.bash
   ros2 launch aidin_rby1_teleop_bringup rs_multi.launch.py
   ```

4. **원격제어 PC — 입력 디바이스 인터페이스**

   MANUS 글러브 · VIVE 트래커 3개 · CloudXR VR 스트리밍 · MCAP 로거를 한 번에 실행합니다.

   ```bash
   cd ~/Workspace/lgit_ws && source install/setup.bash
   ros2 launch aidin_rby1_teleop_bringup teleop_bringup.launch.py
   ```

   CloudXR SDK 가 설치되지 않은 PC 에서는 VR 스트리밍을 제외합니다.

   ```bash
   ros2 launch aidin_rby1_teleop_bringup teleop_bringup.launch.py use_vr_stream:=false
   ```

   | 인자 | 기본값 | 설명 |
   | --- | --- | --- |
   | `use_vr_stream` | `true` | CloudXR VR 스트리밍 스택 실행 여부 |
   | `endpoint_ip` | `""` (자동) | 헤드셋이 접속할 이 PC 의 IP |
   | `output_dir` | `~/data/mcap` | MCAP 출력 디렉터리 |
   | `config` | `ros2_mcap_recorder` 의 `topics.yaml` | 녹화 대상 토픽 목록 |
   | `image_format` | `compressed` | 컬러 이미지 녹화 형식 (`raw` / `compressed`) |

   로거·VR 만 따로 띄우려면 `logger.launch.py` / `teleop_interface.launch.py` 를 개별 실행할 수 있습니다 ([Quick Start](../../introduction/quick-start/) 참고).


## 사전 점검 체크리스트

- [ ] 모든 트래커 추적 상태 정상 (LED 녹색)
- [ ] MANUS 글로브 양손 연결됨
- [ ] RBY1 안전 영역 내에 사람/장애물 없음
- [ ] E-Stop 위치 확인

부팅 후 네트워크·토픽·장비가 정상인지 항목별로 확인하려면
[시스템 초기화 체크리스트](../system-check/)로 진행하세요.
