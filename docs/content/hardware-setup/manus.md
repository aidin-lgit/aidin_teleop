+++
title = "MANUS Glove 셋업"
weight = 3
+++

## 페어링

1. MANUS 동글을 텔레옵 PC USB 포트에 연결
2. MANUS Core 또는 자체 브릿지 실행
3. 좌/우 글로브 전원 ON 후 자동 페어링 확인

## 캘리브레이션

캘리브레이션 절차(자세 시퀀스, 검증 단계, 재캘리 기준 등)는 MANUS 공식 기술 문서를 따라 진행합니다.

- [MANUS Prime 3 Mocap — Calibration (공식 가이드)](https://docs.manus-meta.com/2.3.0/Products/Prime%203%20Mocap/Calibration/)

캘리브레이션이 끝나면 MANUS Core 가 생성한 결과 파일을 워크스페이스 설정 디렉토리에 복사하여 ROS 2 브릿지가 동일한 프로파일을 사용하도록 합니다.


