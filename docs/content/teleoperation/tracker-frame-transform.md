+++
title = "트래커-로봇 좌표 변환"
weight = 2
+++

## 핵심 아이디어

본 시스템은 **별도의 트래커-로봇 캘리브레이션 절차를 수행하지 않습니다.**
대신 작업자의 허리에 부착한 VIVE Tracker 를 RBY1 의 `base_footprint` 와 **고정 변환**으로 묶어
모든 입력을 로봇 베이스 기준으로 즉시 매핑합니다.

```
T_world_to_base   = T_world_to_waistTracker · T_waistTracker_to_base
T_left_ee_target  = T_world_to_base⁻¹ · T_world_to_leftWristTracker · T_offsetL
T_right_ee_target = T_world_to_base⁻¹ · T_world_to_rightWristTracker · T_offsetR
```

- `T_waistTracker_to_base` : 허리 트래커 부착 지점과 RBY1 `base_footprint` 간 **고정 오프셋** (설정 파일에 사전 정의)
- `T_offsetL`, `T_offsetR` : 손목 트래커와 작업자 손바닥 중심 간 오프셋 (글로브 형상 기준 상수)

## 장점

- 작업자가 위치를 옮겨도 추가 보정 불필요 (베이스가 작업자를 따라옴)
- SteamVR 좌표계의 절대 원점이 무의미해지므로 베이스 재배치마다 재캘리브레이션할 필요가 없음
- 셋업 시간이 짧고 재현성이 높음

## 설정 위치

`aidin_rby1_teleop/config/tracker_to_robot.yaml`

```yaml
waist_tracker:
  parent_frame: base_footprint
  translation: [x, y, z]   # meters
  rotation_rpy: [r, p, y]  # radians
left_wrist_offset:
  translation: [...]
  rotation_rpy: [...]
right_wrist_offset:
  translation: [...]
  rotation_rpy: [...]
```

## 점검 방법

- `ros2 run tf2_tools view_frames` 로 TF 트리에 허리 트래커 ↔ `base_footprint` 가 고정 변환으로 연결되는지 확인
- RViz 에서 작업자가 한 자리에 가만히 있을 때 손목 트래커 TF 가 떨리지 않는지 확인 (jitter 가 크면 베이스 스테이션 시야 재검토)

> *TODO: 실제 사용 중인 오프셋 측정 방법 / 허용 오차 / 부착 픽스처 사진*
