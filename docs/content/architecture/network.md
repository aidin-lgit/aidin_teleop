+++
title = "데이터 플로우 / 네트워크"
weight = 3
+++

## 데이터 플로우 (런타임)

```
VIVE Tracker ─┐
MANUS Glove ──┼─▶ Teleop PC ──ROS2──▶ RBY1 + AIDIN Hand
Meta Quest  ──┘                 │
                                └─▶ MCAP Recorder ─▶ /data/raw/*.mcap
```

## 데이터 플로우 (오프라인)

```
*.mcap ──▶ MCAP→HDF5 변환기 ──▶ *.hdf5 ──▶ Data Loader ──▶ 학습 ──▶ Checkpoint
                                                                      │
                                                                      ▼
                                                              ROS2 추론 노드 ──▶ RBY1
```

## 네트워크 토폴로지

- **유선**: Teleop PC ↔ RBY1 (Gigabit, 고정 IP 권장)
- **Wi-Fi (5GHz/Wi-Fi 6)**: Meta Quest 스트리밍 전용 SSID 분리 권장
- **USB**: VIVE 베이스 동기화(케이블), MANUS 동글

## 시스템 하드웨어 간 네트워크 기본 설정

| 장비 | IP 주소 |
| --- | --- |
| RBY1 RPC | `192.168.30.1` |
| RBY1 Lidar 1 | `192.168.30.10` |
| RBY1 Lidar 2 | `192.168.30.11` |
| RBY1 UPC | `192.168.2.21` |
| Teleop PC | `192.168.2.31` |

- **RBY1 RPC**: Rainbow Robotics 측 내부 실시간 제어기.
- **RBY1 UPC**: 사용자 PC 와 직접 통신하는 RBY1 측 상위 PC.
- **Teleop PC**: 사용자(작업자) 측 워크스테이션. ROS 2 노드와 텔레오퍼레이션 / 학습 파이프라인이 실행되는 메인 호스트

> 모든 호스트는 같은 `192.168.2.0/24` 서브넷에 고정 IP 로 묶이며, ROS 2 DDS 통신이 정상 동작하려면 양 호스트의 `ROS_DOMAIN_ID`(default: 1) 가 동일해야 합니다.
