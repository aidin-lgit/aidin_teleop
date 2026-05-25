+++
title = "부팅 시퀀스"
weight = 1
+++

## 권장 순서

1. **로봇 측**
   - RBY1 전원 ON → 자가진단 통과 확인
   - AIDIN Hand 전원 ON → 초기 위치 확인
2. **추적 측**
   - VIVE Lighthouse 베이스 ON
   - SteamVR 실행 후 3개 트래커 (좌 손목 / 우 손목 / 허리) 모두 녹색 확인
   - MANUS Glove ON, Core 에서 페어링 확인
   - Meta Quest 부팅 후 스트리밍 앱 실행
3. **소프트웨어**
   ```bash
   cd ~/aidin_ws && source install/setup.bash
   ros2 launch aidin_rby1_teleop bringup.launch.py
   ```
4. **텔레옵 노드 활성화**
   ```bash
   ros2 launch aidin_rby1_teleop teleop.launch.py
   ```

## 사전 점검 체크리스트

- [ ] 모든 트래커 추적 상태 정상
- [ ] MANUS 글로브 양손 캘리브레이션 로드됨
- [ ] Quest 스트리밍 영상이 PC 측에 정상 표시됨
- [ ] RBY1 안전 영역 내에 사람/장애물 없음
- [ ] E-Stop 위치 확인
