+++
title = "VIVE Tracker 셋업"
weight = 2
+++

## 사전 준비 — Steam / SteamVR

VIVE Tracker 인식·페어링·자세 스트리밍은 모두 **SteamVR** 을 통해 이루어지므로, Teleop PC 에 다음이 미리 갖춰져 있어야 합니다.

1. [Steam](https://store.steampowered.com/about/) 설치 및 Steam 계정 로그인
2. Steam 라이브러리에서 [SteamVR](https://store.steampowered.com/app/250820/SteamVR/) 설치
3. SteamVR 실행 후 정상 구동 확인 (상태 창의 베이스/HMD/트래커 슬롯이 표시되어야 함)

원격제어를 시작하기 전 SteamVR 가 항상 실행 중이어야 하며, Steam 로그아웃 / 자동 업데이트 등으로 SteamVR 가 종료되면 트래커 토픽이 끊기므로 주의하세요.

## 베이스 스테이션 배치

- 최소 2개, 권장 4개 (대각선 배치, 약 2~2.5m 높이, 추적 영역을 안쪽으로 향하게)
- 베이스 간 시야가 서로 직접 보이지 않도록 함 (Lighthouse 2.0 채널 충돌 회피)

## 트래커 페어링

1. 페어링을 시도할 트래커 준비 및 동글 USB 연결
2. SteamVR 실행 후 **장치 → 페어링** 메뉴 진입
3. 트래커 전원 ON → LED 청색 깜빡임 상태에서 페어링

## 부착 위치

| 트래커 | 부착 위치 | 용도 |
| --- | --- | --- |
| Left Wrist | 작업자 좌 손목 | 좌 EE 목표 |
| Right Wrist | 작업자 우 손목 | 우 EE 목표 |
| Waist | 작업자 허리 | RBY1 `base_footprint` 와 고정 변환 |

{{% notice style="warning" title="허리 트래커 부착 방향" %}}
**허리 트래커는 LED(상태 표시등)가 위쪽을 향하도록 부착해야 합니다.** 본 시스템은 별도의 트래커-로봇 캘리브레이션 없이 허리 트래커의 로컬 좌표계를 기준으로 변환을 적용하므로, 부착 방향이 다르면 사용자가 앞으로 손을 뻗어도 로봇이 옆/뒤로 움직이는 등 비정상 동작이 발생합니다.
{{% /notice %}}

![허리 트래커 부착 방향 참고](/images/vive_waist.png?width=300px)


## 동작 점검

- SteamVR Status 창에서 세 트래커 모두 녹색 상태 확인
- 트래커 자세 출력 값이 jitter / dropout 발생 시 베이스 위치 재조정

> 본 시스템은 별도의 **트래커-로봇 캘리브레이션 절차가 없습니다**. 장착한 허리 트래커가 로봇 베이스 기준점이 되며, 변환 관계는 설정 파일의 고정 오프셋으로 처리됩니다.
