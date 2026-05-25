+++
title = "AIDIN Teleoperation 시스템 문서"
type = "home"
+++


휴머노이드 로봇 **RBY1** 과 **AIDIN Robot Hand** 를 결합한 원격제어(Teleoperation) 및
Physical AI End-to-End 학습·추론 시스템의 공식 매뉴얼입니다.

본 문서는 다음 파이프라인을 다룹니다.

1. **원격제어** — VIVE Tracker · MANUS Glove · Meta Quest 를 이용한 휴머노이드 로봇의 원격 제어
2. **데이터 수집** — ROS2 통신 메시지들의 MCAP 데이터 기록 후 HDF5 또는 LeRobot으로 변환
3. **학습** — Diffusion Policy 및 다양한 모방학습 모델 학습
4. **추론** — 학습된 정책을 로봇에 배포하여 자율 작업 수행

## 빠르게 둘러보기

| 섹션 | 내용 |
| --- | --- |
| [시작하기](introduction/) | 시스템 개요, 요구사항, Quick Start |
| [시스템 구성](architecture/) | 하드웨어·소프트웨어 아키텍처 |
| [하드웨어 셋업](hardware-setup/) | 로봇·트래커·글로브·HMD 셋업 |
| [소프트웨어 설치](software-install/) | 의존성, 워크스페이스, 빌드 |
| [원격제어](teleoperation/) | 실행 절차 및 운영 가이드 |
| [데이터 수집](data-collection/) | MCAP → HDF5 파이프라인 |
| [학습](training/) | Data Loader · 모델 · 학습 |
| [추론](inference/) | 모델 배포 및 자율 작업 |
| [튜토리얼](tutorial/) | End-to-End 예제 |
| [문제 해결](troubleshooting/) | FAQ 및 트러블슈팅 |
| [참고](reference/) | 외부 링크·API |
