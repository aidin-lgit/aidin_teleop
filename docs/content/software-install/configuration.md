+++
title = "설정 파일 / 환경 변수"
weight = 3
+++

## 환경 변수

| 변수 | 용도 | 예시 |
| --- | --- | --- |
| `AIDIN_WS` | 워크스페이스 루트 | `~/aidin_ws` |
| `AIDIN_DATA_DIR` | MCAP/HDF5 데이터 저장 루트 | `/data/aidin` |
| `RBY1_IP` | RBY1 본체 IP | `192.168.1.10` |
| `ROS_DOMAIN_ID` | ROS2 도메인 분리 | `42` |

`~/.bashrc` 또는 `~/aidin_ws/setup_env.bash` 에 export 추가.

## 주요 설정 파일 위치

| 파일 | 역할 |
| --- | --- |
| `aidin_rby1_teleop/config/teleop.yaml` | 텔레옵 노드 파라미터 |
| `aidin_rby1_teleop/config/tracker_to_robot.yaml` | 허리 트래커 → `base_footprint` 고정 변환 |
| `aidin_rby1_teleop/config/retarget_left.yaml` | 좌 손목/손가락 리타게팅 매핑 |
| `aidin_rby1_teleop/config/retarget_right.yaml` | 우 손목/손가락 리타게팅 매핑 |
| `aidin_rby1_teleop/config/record.yaml` | 기록 대상 토픽 목록 (MCAP) |

> *TODO: 실제 yaml 키와 기본값 표 추가, 환경 변수와 yaml 파라미터 우선순위 명시*
