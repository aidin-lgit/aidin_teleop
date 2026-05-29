+++
title = "부팅 시퀀스"
weight = 1
+++

## 권장 순서

1. **로봇 플랫폼**
   - RBY1 RPC 전원 ON
   - RBY1 UPC 전원 ON
   - AIDIN Hand 전원 ON
   - 로봇 제어기 실행
   ```bash
   ros2 launch aidin_rby1_controller bimanual_controller.launch.py
   ```

   {{% notice style="warning" title="Homing 자세 이동 시 안전 확인" %}}
   위 launch 실행 시 로봇이 [`initial_positions.yaml`](https://github.com/aidin-lgit/aidin_rby1_description/blob/main/model/urdf/initial_positions.yaml) 에 정의된 초기화 자세로 즉시 Homing 합니다. 양 팔·허리·헤드가 모두 동시에 움직이므로 **실행 전 로봇 주변과 이동 경로상에 사람·장애물이 없는지 반드시 확인**하고, E-Stop 이 손에 닿는 위치에 있는지 점검하세요.
   {{% /notice %}}

2. **트래킹 장비**
   - VIVE Lighthouse 베이스 ON
   - SteamVR 실행 후 3개 트래커 (좌 손목 / 우 손목 / 허리) 모두 녹색 확인

     ![SteamVR 트래커 상태](/images/streamvr.png?width=300px)

   - MANUS Glove ON, Core 에서 페어링 확인
   - Meta Quest 부팅 후 브라우저 앱 실행 (Ego-view 원격제어 모드 시)
3. **원격제어 PC**
   - 트래킹 장치(카메라, 트래커, 햅틱글러브) 인터페이스 실행
   ```bash
   cd ~/Workspace/lgit_ws && source install/setup.bash
   ros2 launch aidin_rby1_teleop_bringup teleop_interface.launch.py
   ```


## 사전 점검 체크리스트

- [ ] 모든 트래커 추적 상태 정상 (LED 녹색)
- [ ] MANUS 글로브 양손 연결됨
- [ ] RBY1 안전 영역 내에 사람/장애물 없음
- [ ] E-Stop 위치 확인
