+++
title = "MANUS Glove 셋업"
weight = 3
+++

## 페어링

1. MANUS 동글을 Teleop PC USB 포트에 연결
2. MANUS Core 또는 자체 브릿지 실행
3. 좌/우 글로브 전원 ON 후 자동 페어링 확인

## USB 권한 — 라이선스 인식용 udev rule

Linux 환경에서 MANUS SDK 가 라이선스 동글을 인식하려면 해당 USB 장치에 대한 권한이 일반 사용자에게 열려 있어야 합니다. MANUS SDK 실행 시 다음과 같은 경고가 뜨면 udev rule 이 누락된 상태입니다.

```
[warning] No compatible license found. Please connect a license with the SDK component.
```

해결 절차:

1. `/etc/udev/rules.d/99.manus.rules` 파일을 생성하고 다음 내용을 작성합니다.

   ```
   SUBSYSTEM=="hidraw", ATTRS{idVendor}=="3325", MODE="0666"
   SUBSYSTEM=="usb",    ATTRS{idVendor}=="3325", MODE="0666"
   SUBSYSTEM=="usb",    ATTRS{idVendor}=="1915", MODE="0666"
   ```

2. udev 규칙을 reload 하고 현재 연결된 장치에 다시 적용합니다.

   ```bash
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   ```

3. 동글이 시스템에서 인식되는지 확인합니다.

   ```bash
   lsusb | grep 3325
   ```

   해당 라인이 표시되지 않으면 USB 포트 / 케이블 / 동글 연결 상태를 먼저 점검하세요.

## 캘리브레이션

캘리브레이션 절차(자세 시퀀스, 검증 단계, 재캘리 기준 등)는 MANUS 공식 기술 문서를 따라 진행합니다.

- [MANUS Prime 3 Mocap — Calibration (공식 가이드)](https://docs.manus-meta.com/2.3.0/Products/Prime%203%20Mocap/Calibration/)

캘리브레이션이 끝나면 MANUS Core 가 생성한 결과 파일을 워크스페이스 설정 디렉토리에 복사하여 ROS 2 브릿지가 동일한 프로파일을 사용하도록 합니다.


