+++
title = "하드웨어 구성"
weight = 1
+++

## RBY1 휴머노이드

![RBY1 Model A](/images/rby1-a.png?width=300px)

[Rainbow Robotics](https://www.rainbow-robotics.com/) 의 휴머노이드 플랫폼. 본 시스템은 RBY1 의 상체(토크 7-DOF × 2 암 + 토르소 + 헤드)에 자사 [AIDIN Hand](#aidin-hand) 를 부착하여 양손 매니퓰레이션을 수행합니다.

### 모델 라인업

| 항목 | Model A | Model M |
| --- | --- | --- |
| 자유도 (DOF) | 24 (휠 2 + 토르소 6 + 양팔 14 + 헤드 2) | 26 (휠 4 + 동일 상체 구성) |
| 모바일 베이스 | 2-wheel (51 kg) | 4-wheel (90 kg) |
| 전체 무게 | 131 kg | 170 kg |

### 외형 / 구조

- **크기**: 600 × 690 × 1,400 mm (W × D × H)
- **상체**: 38 kg (양팔 11 kg × 2 + 토르소 16 kg)
- **하체**: 42 kg
- **외장**: 알루미늄

### 매니퓰레이션

- **암 구성**: 7-DOF × 2 (양팔)
- **암 리치**: 600 mm (손목까지) + 핸드
- **페이로드**: 3 kg / 암
- **반복 정밀도**: < ±0.05 mm
- **관절 최대 속도**: 180 °/s (앵클 롤 등 일부 관절 120 °/s)
- **관절 가동범위 예시**: 숄더 피치 −180° ~ +180°, 엘보 피치 −150° ~ 0°

### 모빌리티

- **최대 이동 속도**: 1.5 m/s (모바일 베이스)

### 전원

- **배터리**: 50 V, 25 Ah (1,270 Wh)
- **공급 전압**: 48 VDC

### 운용 환경

- **주변 온도**: 최대 40 °C

### 참고

- 공식 하드웨어 문서: <https://rainbowrobotics.github.io/rby1-dev/hardware/overview.html>


## AIDIN Hand

[AIDIN ROBOTICS](https://www.aidinrobotics.co.kr/robotic-hand) 의 자사 다지 로봇 핸드. RBY1 의 양 손목에 장착되어 본 시스템의 엔드이펙터로 동작합니다.

### AIDIN Hand Gen1

![AIDIN Hand Gen1](/images/aidin_hand_gen1.png?width=300px)

#### 주요 특징

- **사람 손 비례**의 컴팩트한 사이즈
- 손끝 **Force/Torque 센서 내장** 
- **링크 기반 구조** → 높은 제어 정밀도 및 힘 전달 효율

#### Specification

| Index | Unit | Value |
| --- | --- | --- |
| Grasping mode | Mode | Power Mode (cylindrical, spherical, etc.) / Precision Mode (pinch, tripod, etc.) |
| Degree of Freedom (Finger) | DoF | 3 |
| Degree of Freedom (Hand) | DoF | 15 |
| Finger-tip force | N | 20 |
| Payload | kg | 15 |
| Size | mm | 291 × 112 × 120 |
| Weight | kg | 1.3 |
| Finger-tip sensor | EA | 5 (Miniature 6-axis F/T Sensor, AFT50-D15) |

출처: <https://www.aidinrobotics.co.kr/robotic-hand>

## VIVE Tracker

![HTC VIVE Tracker 3.0](/images/vive_tracker.png?width=200px)

- 모델: HTC VIVE Tracker 3.0
- 부착 위치: 양 손목 (×2) + 허리 (×1)
- 베이스 스테이션: HTC VIVE Base Station 2.0 ×3

## MANUS Glove

[MANUS Meta](https://www.manus-meta.com/) 의 데이터/햅틱 글로브. 작업자 손가락 관절을 실시간으로 측정하여 [AIDIN Hand](#aidin-hand) 에 리타게팅하고, 햅틱 모듈로 그립 피드백을 전달합니다.

![MANUS Metagloves Pro Haptic](/images/manus_haptic.webp?width=200px)

#### Specification

| Index | Value |
| --- | --- |
| Signal latency | 30 ms (wired), 50 ms (wireless) |
| Finger sensor type | 5 EMF sensors with 6 DOF tracking |
| Sensor sample rate | 120 Hz |
| Haptic module type | LRA vibration motor |
| Battery duration | Up to 3 hours (swappable) |
| Charging time | 2 hours (USB-C) |
| Weight | 178 g |
| Wired communication | USB-C |
| Wireless communication | Bluetooth Low Energy 5 |
| Wireless range | Up to 15 m |
| Textile sizes | M, L |


## Meta Quest

[Meta](https://www.meta.com/) 의 VR HMD. 본 시스템에서는 1인칭 시점 영상과 head pose 를 텔레옵 PC 로 송신하는 용도로 사용합니다.

### Meta Quest 3

![Meta Quest 3](/images/meta_quest.webp?width=200px)

#### Specification

| Index | Value |
| --- | --- |
| Processor | Qualcomm Snapdragon XR2 Gen 2 |
| Storage | 512 GB |
| RAM (DRAM) | 8 GB |
| Display (per eye) | 2064 × 2208 px (4K+ Infinite Display) |
| Pixel density | 25 PPD / 1218 PPI |
| Refresh rate | 72 / 90 / 120 Hz |
| Field of view | 110° (H) × 96° (V) |
| Optics | Pancake lens |
| Passthrough | 2× RGB 카메라 (18 PPD) + depth projector |
| Hand tracking | Direct Touch (4× IR + 2× RGB 카메라, CV + ML 하이브리드) |
| Controller | Meta Quest Touch Plus (TruTouch 가변 햅틱) |
| Weight | 515 g (안면 인터페이스 제외) |
| Battery life | 약 2.2 시간 (게이밍 2.4 / 미디어 2.9 / 생산성 1.5) |
| Charging | 18 W USB-C 어댑터, 약 2.3 시간 |
| Wired I/O | USB-C |

출처: <https://www.meta.com/kr/quest/quest-3/>

