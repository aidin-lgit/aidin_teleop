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
- **PTP/NTP**: 모든 입력 소스 간 시간 동기 (학습 데이터 정합성에 직결)

> *TODO: 실제 IP/포트/방화벽 규칙 표 추가*
