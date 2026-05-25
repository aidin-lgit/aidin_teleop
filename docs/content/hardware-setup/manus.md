+++
title = "MANUS Glove 셋업"
weight = 3
+++

## 페어링

1. MANUS 동글을 텔레옵 PC USB 포트에 연결
2. MANUS Core 또는 자체 브릿지 실행
3. 좌/우 글로브 전원 ON 후 자동 페어링 확인

## 캘리브레이션

1. MANUS Core 의 글로브 캘리브레이션 마법사 실행
2. 손 펴기 / 주먹 / 핀치 등 안내된 자세 순서대로 수행
3. 캘리브레이션 결과 파일을 워크스페이스 설정 디렉토리에 복사

## ROS2 브릿지 점검

```bash
ros2 topic echo /manus/left/joint_states
ros2 topic echo /manus/right/joint_states
```

각 토픽이 60Hz 내외로 안정적으로 들어오는지 확인.

> *TODO: 사용 중인 글로브 모델·캘리브레이션 파일 위치·리타게팅 맵핑 표 추가*
