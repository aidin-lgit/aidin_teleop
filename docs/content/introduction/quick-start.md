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

RBY1 UPC 에 SSH 접속해 `aidin_rby1_controller` 의 양팔 컨트롤러를 실행합니다. Teleop PC 의 `aidin_rby1_teleop_bringup/robot_launch.sh` 를 쓰면 SSH 까지 한 번에 처리됩니다.

```bash
# Teleop PC 에서 로봇 제어기 실행
ssh nvidia@192.168.2.21
cd ~/Workspace/aidin_ws 
source install/setup.bash
ros2 launch aidin_rby1_controller bimanual_controller.launch.py
```

동작 확인:

```bash
ros2 topic list | grep aidin_rby1
ros2 topic hz /joint_states
ros2 topic hz /torso_pose_broadcaster/pose
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

세 토픽이 모두 60 Hz 내외로 들어오면 정상입니다. 트래커 인식·페어링이 안 되면 [VIVE Tracker 셋업](../../hardware-setup/vive-tracker/) 을 다시 확인하세요.

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

## 5. VR 스트리밍 (Meta Quest)

ZED → HMD 영상 송출 + HMD/양손 pose 발행 노드를 단독 실행합니다.

```bash
ros2 run aidin_rby1_vr_teleop teleop_stream.py
```

이어서 HMD 브라우저에서 Teleop PC 스트리밍 서버 주소로 접속해 **VR 시작하기** 버튼을 누르면 머리 자세가 송신되고 영상이 수신됩니다 (절차: [원격제어 → 머리 카메라 제어 시 이미지 수신 방법](../../teleoperation/operation/#머리-카메라-제어-시-이미지-수신-방법)).

동작 확인:

```bash
ros2 topic hz /hmd_pose
ros2 topic hz /stereo/left/color/image_raw
```

영상 지연이 심하면 `vuer_image_scale` / `vuer_jpeg_quality` / `vuer_fps` 파라미터를 낮춰 보세요 ([상세](../../teleoperation/operation/#머리-카메라-제어-시-이미지-수신-방법)).

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

`./logs/session_<ts>/episode_1/episode_1_0.mcap` 가 생성되었는지 확인합니다. 자세한 사용법은 [데이터 수집 → 녹화 절차](../../data-collection/recording/) 와 [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) README 를 참고하세요.

## 7. 통합 실행

위 모듈들이 모두 정상 동작하는 것이 확인되면, 두 줄로 전체 인터페이스를 한 번에 띄울 수 있습니다.

```bash
# Terminal 1 — 모든 입력 디바이스 인터페이스 (카메라 + MANUS + VIVE + VR + 로거)
ros2 launch aidin_rby1_teleop_bringup teleop_bringup.launch.py

# Terminal 2 — Teleop 제어 (양팔·허리 + 손가락 + (옵션) 머리)
ros2 launch aidin_rby1_teleop_bringup teleop_control.launch.py vr_control:=true
```

데이터 수집부터 학습·추론까지 전 과정 시연은 [튜토리얼 → End-to-End: Pick & Place](../../tutorial/end-to-end-pick-and-place/) 를 참고하세요.
