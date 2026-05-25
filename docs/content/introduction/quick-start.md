+++
title = "Quick Start"
weight = 4
+++

> 이 문서는 시스템이 이미 설치·셋업되어 있다는 전제 하에 5분 안에 텔레옵을 한 번 돌려보기 위한 절차입니다.
> 처음 설치하는 경우 [하드웨어 셋업](../../hardware-setup/) → [소프트웨어 설치](../../software-install/) 를 먼저 진행하세요.

## 1. 하드웨어 ON

1. RBY1 전원 ON
2. AIDIN Hand 전원 ON
3. VIVE Lighthouse 베이스 스테이션 ON
4. MANUS Glove 전원 ON
5. Meta Quest 부팅 후 스트리밍 앱 실행

## 2. 텔레옵 PC 부팅

```bash
cd ~/aidin_ws
source install/setup.bash
```

## 3. Launch

```bash
ros2 launch aidin_rby1_teleop teleop.launch.py
```

## 4. 작업자 착용

- 양 손목 + 허리 VIVE Tracker 부착
- MANUS Glove 양손 착용
- Meta Quest HMD 착용

## 5. 텔레옵 시작

운영용 GUI 또는 키 입력으로 텔레옵 활성화 후 작업 수행.

## 6. 데이터 기록 (선택)

별도 터미널에서

```bash
ros2 launch aidin_rby1_teleop record.launch.py episode_name:=demo_001
```

> *TODO: 실제 launch 파일명/파라미터에 맞게 보정*
