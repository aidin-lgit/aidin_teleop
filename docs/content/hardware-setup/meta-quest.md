+++
title = "Meta Quest 셋업"
weight = 4
+++

## 초기 설정

1. Quest 부팅 후 동일 Wi-Fi (5GHz / Wi-Fi 6) 접속
2. 개발자 모드 활성화 (Meta Quest 개발자 계정 필요)
3. `adb` 로 PC 와 연결 확인

```bash
adb devices
```

## 스트리밍 앱 설치

- *TODO: 자사/오픈소스 스트리밍 앱 APK 경로*
- `adb install` 로 APK 설치

```bash
adb install -r aidin_teleop_streamer.apk
```

## 시점 영상 송신

- Quest 카메라/렌더 영상을 Teleop PC로 송신
- 토픽 예시: `/quest/left_eye/image_raw`, `/quest/right_eye/image_raw`

## Head Pose 송신

- IMU/Inside-out 추적 기반 head pose 를 ROS2 토픽으로 송신
- 토픽 예시: `/quest/head_pose`

> *TODO: 실제 패키지 이름·토픽·캘리브레이션 절차*
