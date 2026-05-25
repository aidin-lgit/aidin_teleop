+++
title = "데이터 포맷: MCAP → HDF5"
weight = 1
+++

## 왜 MCAP + HDF5 인가

| 포맷 | 역할 | 이유 |
| --- | --- | --- |
| **MCAP** | Raw 기록 | ROS2 네이티브, 토픽 다양성·시간 정렬·스트리밍 친화 |
| **HDF5** | 학습용 데이터셋 | 랜덤 액세스, 청크/압축, NumPy/PyTorch 친화 |

원칙: **원본 MCAP 은 절대 수정하지 않는다.** 변환된 HDF5 에서만 가공/증강을 수행.

## 기록 대상 토픽 (예시)

| 토픽 | 메시지 | 비고 |
| --- | --- | --- |
| `/rby1/joint_states` | `sensor_msgs/JointState` | 로봇 관절 상태 |
| `/aidin_hand/{left,right}/joint_states` | `sensor_msgs/JointState` | 손 관절 상태 |
| `/teleop/cmd/{left,right}_wrist_pose` | `geometry_msgs/PoseStamped` | 목표 EE 포즈 |
| `/teleop/cmd/{left,right}_hand_joint` | `sensor_msgs/JointState` | 손가락 명령 |
| `/quest/{left,right}_eye/image_raw` | `sensor_msgs/Image` | 1인칭 영상 |
| `/quest/head_pose` | `geometry_msgs/PoseStamped` | HMD 자세 |
| `/vive/{left_wrist,right_wrist,waist}` | `geometry_msgs/PoseStamped` | 트래커 raw |
| `/teleop/engaged` | `std_msgs/Bool` | 텔레옵 활성 여부 |

> 실제 토픽 이름은 `aidin_rby1_teleop/config/record.yaml` 의 화이트리스트 참조.

## HDF5 스키마 (예시)

```
episode_0001.hdf5
├── /meta/
│   ├── episode_id            : str
│   ├── operator              : str
│   ├── task                  : str
│   ├── start_time_ns         : int64
│   └── duration_s            : float32
├── /obs/
│   ├── joint_pos             : (T, J)     float32
│   ├── joint_vel             : (T, J)     float32
│   ├── hand_left_joint_pos   : (T, JL)    float32
│   ├── hand_right_joint_pos  : (T, JR)    float32
│   ├── wrist_left_pose       : (T, 7)     float32  (xyz + quat)
│   ├── wrist_right_pose      : (T, 7)     float32
│   ├── head_pose             : (T, 7)     float32
│   ├── image_left            : (T, H, W, 3) uint8 (chunked, compressed)
│   └── image_right           : (T, H, W, 3) uint8
├── /action/
│   ├── wrist_left_target     : (T, 7)     float32
│   ├── wrist_right_target    : (T, 7)     float32
│   ├── hand_left_target      : (T, JL)    float32
│   └── hand_right_target     : (T, JR)    float32
└── /index/
    └── timestamps_ns         : (T,)       int64
```

## 변환 도구

```bash
ros2 run aidin_rby1_teleop mcap_to_hdf5 \
    --input  /data/raw/episode_0001.mcap \
    --output /data/hdf5/episode_0001.hdf5 \
    --config aidin_rby1_teleop/config/mcap_to_hdf5.yaml
```

> *TODO: 실제 노드 이름, 변환 시 다운샘플링·동기화 정책(가장 가까운 타임스탬프 vs 보간) 명시*
