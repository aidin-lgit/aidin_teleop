+++
title = "Quick Start"
weight = 4
+++

> 이 페이지는 시스템을 구성하는 **개별 모듈을 하나씩 켜서 동작을 확인**하는 절차입니다. 통합 실행으로 한 번에 전체 파이프라인을 돌려보고 싶다면 [튜토리얼 → End-to-End: Pick & Place](../../tutorial/end-to-end-pick-and-place/) 를 참고하세요.

## 0. 사전 준비

워크스페이스가 이미 빌드·셋업되어 있다고 가정합니다 (처음이라면 [소프트웨어 설치 → 워크스페이스 구성 및 빌드](../../software-install/workspace/) 먼저 진행).

```bash
# Teleop PC
cd ~/Workspace/lgit_ws && source install/setup.bash
```

각 모듈을 개별 터미널에서 띄우고, 별도 터미널에서 `ros2 topic list` / `ros2 topic hz <topic>` 로 데이터 흐름을 점검합니다.

## 1. 로봇 컨트롤러 (RBY1 UPC)

RBY1 UPC 에 SSH 접속해 `aidin_rby1_controller` 의 양팔 컨트롤러를 실행합니다. Teleop PC 의 `aidin_rby1_teleop_bringup/robot_launch.sh` 를 쓰면 SSH 와 환경변수 통일까지 한 번에 처리됩니다 ([부팅 시퀀스](../../teleoperation/launch-sequence/) 참고).

```bash
# RBY1 UPC 에 접속해 직접 실행하는 경우
ssh nvidia@192.168.2.21
cd ~/Workspace/aidin_ws
source install/setup.bash
ros2 launch aidin_rby1_controller bimanual_controller.launch.py \
    use_wholebody_control:=False \
    use_left_hand:=True use_right_hand:=True
```

본 문서는 `use_wholebody_control:=False` (bimanual IK) 를 기준으로 합니다 — 녹화 토픽 화이트리스트(`topics.yaml`)와 문서 전반의 토픽 표기가 이 구성에 맞춰져 있습니다. 컨트롤러 선택 조합과 wholebody IK 로 쓸 때 함께 맞춰야 할 항목은 [부팅 시퀀스 → Cartesian 컨트롤러 선택](../../teleoperation/launch-sequence/#cartesian-컨트롤러-선택) 을 참고하세요.

MANUS 손가락 제어를 쓰려면 `use_left_hand` / `use_right_hand` 를 위와 같이 `True` 로 지정합니다.

동작 확인:

```bash
ros2 topic list | grep aidin_rby1
ros2 topic hz /joint_states
ros2 topic hz /torso_pose_broadcaster/pose

# 활성 Cartesian 컨트롤러가 bimanual 인지 확인
ros2 topic info /aidin_rby1_bimanual_controller/pose_array_command
```

{{% notice style="warning" title="Homing 자세 이동 시 안전 확인" %}}
위 launch 실행 시 로봇이 [`aidin_rby1_description/model/urdf/initial_positions.yaml`](https://github.com/aidin-lgit/aidin_rby1_description/blob/main/model/urdf/initial_positions.yaml) 에 정의된 초기화 자세로 즉시 Homing 합니다. 양 팔·허리·헤드가 모두 동시에 움직이므로 **실행 전 로봇 주변과 이동 경로상에 사람·장애물이 없는지 반드시 확인**하고, E-Stop 이 손에 닿는 위치에 있는지 점검하세요. 자세한 절차는 [부팅 시퀀스](../../teleoperation/launch-sequence/) 참고.
{{% /notice %}}

자세한 launch 인자는 [`aidin_rby1_controller`](https://github.com/aidin-lgit/aidin_rby1_controller) README 를 참고합니다.

## 2. RealSense 카메라

머리·좌·우 RealSense 3대를 한 번에 띄웁니다.

```bash
ros2 launch aidin_rby1_teleop_bringup rs_multi.launch.py
```

{{% notice style="info" title="카메라는 통합 bringup 에 포함되지 않습니다" %}}
`teleop_bringup.launch.py` / `teleop_interface.launch.py` 에는 RealSense 가 **포함되지 않습니다.** 카메라는 항상 이 launch 로 별도 터미널에서 띄우세요 (§7 통합 실행에서도 동일).
{{% /notice %}}

일부 카메라만 쓰거나 시리얼을 바꿔 지정할 수 있습니다.

```bash
ros2 launch aidin_rby1_teleop_bringup rs_multi.launch.py \
    use_head:=false left_serial:=218622274077 right_serial:=218622271511
```

동작 확인:

```bash
ros2 topic list | grep camera
ros2 topic hz /camera/head/color/image_raw
ros2 topic hz /camera/left/color/image_raw
ros2 topic hz /camera/right/color/image_raw
```

각 카메라의 시리얼 번호 매핑·해상도·FPS 변경은 `rs_multi.launch.py` 와 `rs_camera.launch.py` 를 수정하거나 [`aidin_rby1_teleop_bringup`](https://github.com/aidin-lgit/aidin_rby1_teleop_bringup) README 를 참고하세요.

## 3. VIVE 트래커

[SteamVR 사전 준비](../../hardware-setup/vive-tracker/#사전-준비--steam--steamvr) 가 완료된 상태에서 3개 트래커 (좌·우 손목 + 허리) 를 ROS 2 토픽으로 띄웁니다.

```bash
ros2 launch vive_tracker_ros2 vive_bringup.launch.py
```

동작 확인:

```bash
ros2 topic hz /vive_left/vive_tracker_ros/raw_pose
ros2 topic hz /vive_right/vive_tracker_ros/raw_pose
ros2 topic hz /vive_waist/vive_tracker_ros/raw_pose
```

세 토픽이 모두 **100 Hz** 내외로 들어오면 정상입니다. 트래커 인식·페어링이 안 되면 [VIVE Tracker 셋업](../../hardware-setup/vive-tracker/) 을 다시 확인하세요.

이 launch 는 트래커 3개와 함께 CSV 로깅 컨트롤러(`vive_log_controller`) 를 띄웁니다. 좌표계 원점이 될 lighthouse 를 고정하거나 로그 경로를 바꾸려면 인자를 지정합니다.

```bash
ros2 launch vive_tracker_ros2 vive_bringup.launch.py \
    reference_base_serial:=LHB-F8CAA693 log_dir:=/mnt/data/vive_logs
```

{{% notice style="warning" title="허리 트래커는 필수" %}}
허리 트래커는 텔레옵의 기준 좌표계를 캡처하는 데 사용되므로, 없으면 명령이 아예 발행되지 않습니다. 세 토픽 중 `vive_waist` 가 빠지지 않았는지 반드시 확인하세요.
{{% /notice %}}

## 4. MANUS 글러브

MANUS 동글이 연결되고 SDK 라이선스 udev 규칙이 적용된 상태에서 ([MANUS Glove 셋업](../../hardware-setup/manus/) 참고) 글러브 raw 데이터 publisher 를 띄웁니다.

```bash
ros2 run manus_ros2 manus_data_publisher
```

동작 확인:

```bash
ros2 topic hz /manus_glove_0
ros2 topic echo /manus_glove_0 --once
```

`No compatible license found` 경고가 뜨면 [Manus udev 규칙](../../hardware-setup/manus/#usb-권한--라이선스-인식용-udev-rule) 단계를 다시 확인합니다.

## 5. VR 스트리밍 (Meta Quest, CloudXR)

ZED → HMD 영상 송출 + HMD/양손 pose 발행 스택을 단독 실행합니다. NVIDIA CloudXR 런타임 SDK 가 설치되어 있어야 하며, 설치·TLS 인증서 발급 절차는 [`aidin_rby1_vr_teleop`](https://github.com/aidin-lgit/aidin_rby1_vr_teleop) README §2 를 먼저 따릅니다.

```bash
export CLOUDXR_SDK=$HOME/Dev/cloudxr-runtime_6.0.2
ros2 launch aidin_rby1_vr_teleop cloudxr_teleop.launch.py endpoint_ip:=192.168.2.31
```

이 launch 하나로 CloudXR 런타임 · WSS 프록시 · web client · OpenXR 송출 앱 4개 프로세스가 함께 기동됩니다. 이어서 HMD 브라우저에서 접속해 immersive 모드로 진입하면 머리 자세가 송신되고 영상이 수신됩니다 (절차: [원격제어 → 머리 카메라 제어 시 이미지 수신 방법](../../teleoperation/operation/#머리-카메라-제어-시-이미지-수신-방법)).

동작 확인:

```bash
ros2 topic hz /vr/hmd_pose
ros2 topic hz /vr/left_hand_pose
ros2 topic hz /vr/right_hand_pose

# ZED 영상 (camera_location:=local — 결합 stereo 토픽)
ros2 topic hz /zed/zed_node/stereo/color/rect/image
```

`camera_location:=remote` 로 ZED 가 다른 PC 에 붙어 있으면 좌·우 개별 토픽(`/zed/zed_node/{left,right}/color/rect/image`)을 구독해 결합합니다.

CloudXR SDK 가 없는 PC 에서는 이 단계를 건너뛰고, 통합 실행 시 `use_vr_stream:=false` 로 제외하세요. 영상 품질·비트레이트 조정은 [`webclient/README.md`](https://github.com/aidin-lgit/aidin_rby1_vr_teleop/blob/main/webclient/README.md) 를 참고합니다.

## 6. MCAP 로거

녹화 노드를 단독 실행해 토픽 자동 검출이 정상 동작하는지 확인합니다.

```bash
ros2 launch aidin_rby1_teleop_bringup logger.launch.py
```

노드 로그에 `모든 타겟 토픽의 타입을 성공적으로 찾았습니다!` 가 출력되면 정상입니다. 세션·에피소드 제어는 두 개의 `std_srvs/SetBool` 서비스를 호출합니다.

```bash
ros2 service call /dynamic_mcap_recorder/session_control std_srvs/srv/SetBool "{data: true}"
ros2 service call /dynamic_mcap_recorder/record_control  std_srvs/srv/SetBool "{data: true}"
# ... 짧게 녹화 후 ...
ros2 service call /dynamic_mcap_recorder/record_control  std_srvs/srv/SetBool "{data: false}"
ros2 service call /dynamic_mcap_recorder/session_control std_srvs/srv/SetBool "{data: false}"
```

`~/data/mcap/session_<ts>/episode_1/episode_1_0.mcap` 가 생성되었는지 확인합니다.

{{% notice style="warning" title="출력 경로는 실행 방식에 따라 다릅니다" %}}
| 실행 방식 | 기본 출력 경로 |
| --- | --- |
| `ros2 launch ... logger.launch.py` / `teleop_bringup.launch.py` | **`~/data/mcap/`** |
| `ros2 run ros2_mcap_recorder mcap_recorder` (노드 직접 실행) | `./logs/` (실행한 작업 디렉터리 기준) |

launch 로 띄운 세션을 변환할 때는 `--logs-dir ~/data/mcap` 또는 `--session ~/data/mcap/session_<ts>` 를 명시해야 합니다.
{{% /notice %}}

로거 launch 인자로 경로와 이미지 저장 형식을 바꿀 수 있습니다.

```bash
ros2 launch aidin_rby1_teleop_bringup logger.launch.py \
    output_dir:=/mnt/data/teleop_logs image_format:=raw
```

| 인자 | 기본값 | 설명 |
| --- | --- | --- |
| `config` | `ros2_mcap_recorder` 의 `topics.yaml` | 녹화 대상 토픽 목록 |
| `output_dir` | `~/data/mcap` | 세션 디렉터리를 만들 부모 디렉터리 |
| `image_format` | `compressed` | 컬러 이미지 저장 형식. `raw` 는 무손실(용량↑) |

자세한 사용법은 [데이터 수집 → 녹화 절차](../../data-collection/recording/) 와 [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) README 를 참고하세요.

## 7. 통합 실행

위 모듈들이 모두 정상 동작하는 것이 확인되면, 아래 순서로 전체 스택을 띄울 수 있습니다.

```bash
# Terminal 1 — 로봇 컨트롤러 (SSH + 도메인·미들웨어 통일)
cd ~/Workspace/lgit_ws/src/aidin_rby1_teleop/aidin_rby1_teleop_bringup
./robot_launch.sh use_left_hand:=True use_right_hand:=True use_wholebody_control:=False

# Terminal 2 — RealSense 3대 (통합 bringup 에 미포함이므로 별도 실행)
ros2 launch aidin_rby1_teleop_bringup rs_multi.launch.py

# Terminal 3 — 입력 디바이스 인터페이스 (MANUS + VIVE + CloudXR + 로거)
ros2 launch aidin_rby1_teleop_bringup teleop_bringup.launch.py

# Terminal 4 — Teleop 제어 (양팔·허리 + 손가락 + (옵션) 머리)
ros2 launch aidin_rby1_teleop_bringup teleop_control.launch.py vr_control:=true
```

CloudXR SDK 가 없는 PC 라면 Terminal 3 을 `use_vr_stream:=false` 로, Terminal 4 의 `vr_control:=true` 를 제외하고 실행합니다.

Terminal 1 의 launch 인자는 [§1 로봇 컨트롤러](#1-로봇-컨트롤러-rby1-upc) 와 동일합니다.

데이터 수집부터 학습·추론까지 전 과정 시연은 [튜토리얼 → End-to-End: Pick & Place](../../tutorial/end-to-end-pick-and-place/) 를 참고하세요.
