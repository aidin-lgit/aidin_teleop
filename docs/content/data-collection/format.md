+++
title = "데이터 포맷"
weight = 1
+++

## 

| 포맷 | 역할 | 이유 |
| --- | --- | --- |
| **MCAP** | Raw 기록 | ROS2 네이티브, 토픽 다양성·시간 정렬·스트리밍 친화 |
| **HDF5** | 학습용 데이터셋 | 랜덤 액세스, 청크/압축, NumPy/PyTorch 친화 |


## 기록 대상 토픽

녹화 대상은 `ros2_mcap_recorder/config/topics.yaml` 의 토픽 화이트리스트에 따라 결정됩니다. 기본 구성의 토픽은 다음과 같습니다.

### TF / 모델 / 관절 상태

| 토픽 | 메시지 | 비고 |
| --- | --- | --- |
| `/tf` | `tf2_msgs/TFMessage` | 동적 TF |
| `/tf_static` | `tf2_msgs/TFMessage` | 정적 TF (latched) |
| `/robot_description` | `std_msgs/String` | URDF (latched) |
| `/joint_states` | `sensor_msgs/JointState` | 전체 관절 상태 |

### 입력 디바이스

| 토픽 | 메시지 | 비고 |
| --- | --- | --- |
| `/hmd_pose` | `geometry_msgs/PoseStamped` | Meta Quest HMD 머리 자세 |

### 카메라 — RGB-D (Head / Wrist L / Wrist R)

| 토픽 | 메시지 |
| --- | --- |
| `/camera/head/color/camera_info` | `sensor_msgs/CameraInfo` |
| `/camera/head/color/image_raw` | `sensor_msgs/Image` |
| `/camera/head/depth/camera_info` | `sensor_msgs/CameraInfo` |
| `/camera/head/depth/image_rect_raw` | `sensor_msgs/Image` |
| `/camera/left/color/camera_info` | `sensor_msgs/CameraInfo` |
| `/camera/left/color/image_raw` | `sensor_msgs/Image` |
| `/camera/left/depth/camera_info` | `sensor_msgs/CameraInfo` |
| `/camera/left/depth/image_rect_raw` | `sensor_msgs/Image` |
| `/camera/right/color/camera_info` | `sensor_msgs/CameraInfo` |
| `/camera/right/color/image_raw` | `sensor_msgs/Image` |
| `/camera/right/depth/camera_info` | `sensor_msgs/CameraInfo` |
| `/camera/right/depth/image_rect_raw` | `sensor_msgs/Image` |

### 카메라 — Stereo RGB (Head ZED 등)

| 토픽 | 메시지 |
| --- | --- |
| `/stereo/left/color/camera_info` | `sensor_msgs/CameraInfo` |
| `/stereo/left/color/image_raw` | `sensor_msgs/Image` |
| `/stereo/right/color/camera_info` | `sensor_msgs/CameraInfo` |
| `/stereo/right/color/image_raw` | `sensor_msgs/Image` |

### 로봇 상태 — 포즈 브로드캐스터

| 토픽 | 메시지 | 비고 |
| --- | --- | --- |
| `/torso_pose_broadcaster/pose` | `geometry_msgs/PoseStamped` | 토르소 자세 |
| `/left_eef_pose_broadcaster/pose` | `geometry_msgs/PoseStamped` | 좌 EE 자세 |
| `/right_eef_pose_broadcaster/pose` | `geometry_msgs/PoseStamped` | 우 EE 자세 |

### 힘/토크 — FT 브로드캐스터

| 토픽 | 메시지 | 비고 |
| --- | --- | --- |
| `/left_wrist_ft_sensor_broadcaster/wrench` | `geometry_msgs/WrenchStamped` | 좌 손목 6-DoF FT |
| `/right_wrist_ft_sensor_broadcaster/wrench` | `geometry_msgs/WrenchStamped` | 우 손목 6-DoF FT |
| `/left_ft_sensor_broadcaster/wrench` | `sensor_msgs/MultiDOFJointState` | 좌 손가락 FT (5 손가락 묶음) |
| `/right_ft_sensor_broadcaster/wrench` | `sensor_msgs/MultiDOFJointState` | 우 손가락 FT (5 손가락 묶음) |

### 컨트롤러 명령

| 토픽 | 메시지 | 비고 |
| --- | --- | --- |
| `/aidin_rby1_joint_controller/joint_state_command` | `sensor_msgs/JointState` | 관절-공간 직접 명령 |
| `/aidin_rby1_bimanual_controller/pose_array_command` | `geometry_msgs/PoseArray` | 좌 EE / 우 EE / 토르소 명령 |
| `/aidin_rby1_mobile_controller/cmd_vel_unstamped` | `geometry_msgs/Twist` | 모바일 베이스 속도 명령 |
| `/aidin_rby1_mobile_controller/odom` | `nav_msgs/Odometry` | 모바일 베이스 오도메트리 |
| `/aidin_rby1_joint_controller/command_interfaces` | — | 컨트롤러 → 하드웨어 인터페이스 chained 명령 |

### 하드웨어 인터페이스 명령 echo

| 토픽 | 메시지 | 비고 |
| --- | --- | --- |
| `/rby1_hw_node/joint_commands` | `sensor_msgs/JointState` | RBY1 HW 인터페이스에 적용된 관절 명령 |
| `/rby1_hw_node/wheel_velocity_commands` | — | RBY1 HW 인터페이스에 적용된 휠 속도 명령 |

> 토픽을 추가·제거하려면 `ros2_mcap_recorder/config/topics.yaml` 을 수정하고 패키지를 재빌드하세요. 자세한 latched 처리·QoS 자동 검출 동작은 [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) README 를 참고합니다.

## HDF5 스키마

`ros2_mcap_recorder` 의 `convert_session_to_hdf5` / `convert_mcap_to_hdf5` 변환기는 RoboMimic 호환 구조로 HDF5 를 생성합니다. 한 파일에 여러 에피소드가 `demo_0`, `demo_1`, … 그룹으로 누적됩니다.

```text
dataset.hdf5
└── data/                                 # attrs: total_episodes, env_args, ...
    └── demo_0/                           # attrs: num_samples, model_file, task_description, init_state, ...
        ├── actions                       # (N, A)        attrs: layout, sources, source_dims
        ├── states                        # (N, S)        attrs: layout, sources, source_dims
        ├── rewards                       # (N,)
        ├── dones                         # (N,)
        └── obs/
            │  # ── 관절 (proprioception) ──
            ├── joint_arm_L                # (N, J)
            ├── joint_arm_R                # (N, J)
            ├── joint_hand_L               # (N, J)
            ├── joint_hand_R               # (N, J)
            ├── joint_head                 # (N, J)
            ├── joint_torso                # (N, J)
            │     attrs: description, field, joint_names, units, source_topic
            │
            │  # ── 포즈 ──
            ├── pose_eef_L                 # (N, 7)
            ├── pose_eef_R                 # (N, 7)
            ├── pose_torso                 # (N, 7)
            ├── pose_hmd                   # (N, 7)
            │     attrs: description, layout="x,y,z,qx,qy,qz,qw", frame_id, source_topic
            │
            │  # ── 모바일 베이스 ──
            ├── odom_base                  # (N, 7)
            │     attrs: description, layout, frame_id, child_frame_id, source_topic
            │
            │  # ── 힘/토크 ──
            ├── wrench_arm_L               # (N, 6)
            ├── wrench_arm_R               # (N, 6)
            ├── wrench_wrist_L             # (N, 6)
            ├── wrench_wrist_R             # (N, 6)
            │     attrs: description, layout="fx,fy,fz,tx,ty,tz", units, frame_id, source_topic
            │
            │  # ── 카메라 ──
            ├── image_head_color           # (N, H, W, 3)
            ├── image_head_depth           # (N, H, W)
            ├── image_stereo_L             # (N, H, W, 3)
            ├── image_stereo_R             # (N, H, W, 3)
            ├── image_wrist_L_color        # (N, H, W, 3)
            └── image_wrist_R_color        # (N, H, W, 3)
                  attrs: description, height, width, channels, encoding,
                         fps, frame_id, intrinsics_K, distortion, source_topic
```

### 네이밍 규칙

| 모달리티 | 접두사 | 형식 | 예 |
| --- | --- | --- | --- |
| 관절 상태 | `joint_` | `joint_<part>[_<side>]` | `joint_arm_L`, `joint_torso` |
| 포즈 | `pose_` | `pose_<part>[_<side>]` | `pose_eef_R`, `pose_hmd` |
| 힘/토크 | `wrench_` | `wrench_<part>_<side>` | `wrench_wrist_L` |
| 이미지 | `image_` | `image_<location>[_<side>]_<stream>` | `image_head_color`, `image_head_depth` |
| 오도메트리 | `odom_` | `odom_<part>` | `odom_base` |

- `<side>`: `L` / `R`
- `<part>`: `arm`, `hand`, `head`, `torso`, `wrist`, `base`
- `<stream>`: `color`, `depth`, (필요 시) `ir`, `disparity`

### 공통 attribute

모든 dataset 에 다음이 공통으로 부여됩니다.

- `description` *(str)* — 한 줄 설명
- `source_topic` *(str)* — 원본 ROS 2 토픽
- `source_msg_type` *(str)* — 원본 메시지 타입 (예: `sensor_msgs/JointState`)
- `time_unit` *(str)* — `"ns"` / `"s"` 등

> 모달리티별 추가 attribute, `mcap_to_hdf5.yaml` 매핑 규칙(`joint_groups`, `poses`, `wrenches`, `command_streams`, `concat` 으로 `actions`/`states` 생성), LeRobot 출력 포맷은 [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) README §3-3, §3-4 를 참고하세요.

## 변환 도구

변환기는 모두 `ros2_mcap_recorder` 패키지에서 제공됩니다.

```bash
# 단일 MCAP → HDF5
ros2 run ros2_mcap_recorder convert_mcap_to_hdf5 \
    --mcap /data/raw/episode_1_0.mcap \
    --out  /data/hdf5/episode_1.hdf5

# 세션 일괄 → 단일 HDF5 (권장, demo_0 / demo_1 ... 누적)
ros2 run ros2_mcap_recorder convert_session_to_hdf5 \
    --session /data/raw/session_20260729_153000
```

매핑 YAML 을 바꿀 때만 `--config <mcap_to_hdf5.yaml>` 을 지정합니다 (기본값은 패키지 `share/ros2_mcap_recorder/config/mcap_to_hdf5.yaml`). 전체 CLI 옵션은 [후처리 및 검증](../postprocessing/) 을 참고하세요.
