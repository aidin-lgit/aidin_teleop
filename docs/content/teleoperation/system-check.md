+++
title = "시스템 초기화 체크리스트"
weight = 2
+++

[부팅 시퀀스](../launch-sequence/)로 전원·노드를 올린 뒤, **원격제어를 시작하기 전에**
시스템이 정상 상태인지 확인하는 절차입니다. 한 항목이라도 "실패"면 다음 단계로 넘어가지
말고 해당 행의 대응을 먼저 수행하세요.

> 토픽 이름은 환경 설정에 따라 다를 수 있습니다. 정확한 이름은
> [시스템 구성 → 소프트웨어](../../architecture/software/) 와 각 패키지 README 를 참고하세요.

## 1. 전원 · 부팅

| 점검 | 정상 | 실패 시 대응 |
|---|---|---|
| RBY1 (RPC/UPC) 부팅 | 전원 LED 점등, UPC 부팅 완료 | 전원/케이블 확인 후 재부팅 |
| Teleop PC 부팅 | OS 로그인, ROS2 환경 source 완료 | `source /opt/ros/humble/setup.bash` 및 워크스페이스 setup 확인 |
| 베이스 스테이션 / 트래킹 장비 | 전원 ON, 정상 표시 | [하드웨어 셋업](../../hardware-setup/) 참고 |

## 2. 네트워크 통신

```bash
# Teleop PC → RBY1 으로 ping (IP 는 환경에 맞게)
ping <rby1_ip>

# 양쪽 호스트에서 ROS_DOMAIN_ID 가 동일한지 확인
echo $ROS_DOMAIN_ID
```

| 점검 | 정상 | 실패 시 대응 |
|---|---|---|
| `ping <rby1_ip>` | 응답(0% packet loss) | 케이블/IP/스위치 확인. [네트워크 구성](../../architecture/network/) 참고 |
| `ROS_DOMAIN_ID` 일치 | 두 호스트 값이 동일 | 양쪽 환경변수 동일하게 설정 후 노드 재시작 |
| Zenoh 라우터(사용 시) | 라우터 동작 중 | [통신 설정](../../software-install/configuration/) 참고 |

## 3. ROS2 노드 · 토픽

```bash
ros2 node list           # 기대한 노드들이 보이는가
ros2 topic list          # 관절/카메라/트래커 토픽이 보이는가
ros2 topic hz <topic>    # 핵심 토픽이 기대 주기로 들어오는가
```

| 점검 | 정상 | 실패 시 대응 |
|---|---|---|
| 관절 상태 토픽 (예: `/joint_states`) | `ros2 topic hz` 가 기대 주기(예: 수십 Hz) 표시 | 컨트롤러/하드웨어 인터페이스 노드 실행 확인 |
| 카메라 이미지 토픽 | `ros2 topic hz` 가 0 이 아님 | 카메라 드라이버 노드 확인, 인코딩 `rgb8`/`bgr8` 확인 |
| 누락된 노드 없음 | `ros2 node list` 에 기대 노드 모두 존재 | 해당 launch 재실행, 로그 확인 |

## 4. 입력 장비 활성화

| 장비 | 정상 신호 | 실패 시 대응 |
|---|---|---|
| VIVE Tracker | SteamVR Status 에서 트래커 녹색, jitter/dropout 없음 | 트래커 전원·페어링 재확인. [VIVE 셋업](../../hardware-setup/vive-tracker/) |
| MANUS Glove | 좌/우 글로브 인식, 캘리브레이션 파일 로드됨 | 동글/페어링·캘리브레이션 재확인. [MANUS 셋업](../../hardware-setup/manus/) |
| Meta Quest (HMD) | 시점 영상 수신, 지연 허용 범위 | 브라우저/서버 주소 재확인. [Meta Quest 셋업](../../hardware-setup/meta-quest/) |

## 5. 안전 확인

- [ ] 하드웨어 E-Stop 이 손에 닿는 위치에 있고 동작하는가 → [안전 절차](../safety/)
- [ ] 로봇 주변 작업 공간이 비어 있는가
- [ ] 첫 시작은 천천히 / 짧게 진행할 준비가 되었는가

모든 항목이 정상이면 [원격제어 실행 및 조작법](../operation/)으로 진행합니다.
