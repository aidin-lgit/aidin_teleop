+++
title = "시스템 개요"
weight = 1
+++

## 한눈에 보기

```
[Operator] ──VIVE Tracker / MANUS / Meta Quest──▶ [Teleop PC] ──ROS2──▶ [RBY1 + AIDIN Hand]
                                                       │
                                                       └─ MCAP 기록 ─▶ HDF5 변환 ─▶ 학습 ─▶ 추론
```

## 구성 요소 요약

- **휴머노이드 로봇**: Rainbow Robotics **RBY1**
- **엔드이펙터**: AIDIN Robot Hand (자사 제품)
- **상체 위치 추적**: HTC **VIVE Tracker** (양 손목, 허리)
- **손가락 모션**: **MANUS Glove**
- **1인칭 시점 / 머리 자세**: **Meta Quest** 시리즈 (스트리밍 영상 + Head pose)

## End-to-End 파이프라인

1. 작업자가 HMD/글로브/트래커를 착용한 상태로 텔레옵 시작
2. 텔레옵 PC가 입력을 리타게팅하여 RBY1 + AIDIN Hand로 송출
3. 작업이 진행되는 동안 모든 토픽이 MCAP 파일로 기록됨
4. 기록된 MCAP 을 HDF5 로 변환하여 학습 데이터셋 구축
5. Diffusion Policy 등 모방학습 모델을 학습
6. 학습된 정책을 동일한 ROS2 인터페이스로 배포하여 자율 작업 수행

> *TODO: 시스템 전체 다이어그램 이미지 삽입*
