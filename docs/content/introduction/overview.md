+++
title = "시스템 개요"
weight = 1
+++

## 한눈에 보기

### 원격제어 워크플로우

![Teleop Workflow](/images/teleop_workflow.png)

### 데이터 학습 및 추론 워크플로우

![E2E Workflow](/images/e2e_workflow.png)

## 구성 요소 요약

- **휴머노이드 로봇**: Rainbow Robotics **RBY1**
- **엔드이펙터**: AIDIN Robot Hand (자사 제품)
- **상체 위치 추적**: HTC **VIVE Tracker** (양 손목, 허리)
- **손가락 모션**: **MANUS Glove**
- **1인칭 시점 / 머리 자세**: **Meta Quest** 시리즈 (스트리밍 영상 + Head pose)

## End-to-End 파이프라인

1. 로봇 제어기 실행
2. 작업자가 HMD/글로브/트래커를 착용한 상태로 원격제어 시작
2. Teleop PC가 입력을 리타게팅하여 로봇 제어기로 송출
3. 작업이 진행되는 동안 모든 토픽정보를 MCAP 데이터로 저장
4. 기록된 MCAP 을 HDF5 로 변환하여 학습 데이터셋 구축
5. Diffusion Policy 등 모방학습 모델을 학습
6. 학습된 정책을 동일한 ROS2 인터페이스로 배포하여 자율 작업 수행

