+++
title = "RBY1 + AIDIN Hand 조립"
weight = 1
+++

## 조립 순서

1. RBY1 양 손목 플랜지에서 기본 그리퍼 제거 — [Rainbow Robotics 공식 가이드: Detaching Guide](https://rainbowrobotics.github.io/rby1-dev/maintenance/detaching_guide_prev.html) 절차를 따를 것
2. AIDIN Hand 통신 및 전원 케이블 체결
3. 좌/우 통신 케이블을 이더켓 허브에 연결
3. 통신·전원 케이블 라우팅 (어깨→팔뚝→손목)

## 체크리스트

- [ ] 좌/우 손 페어 식별 (좌우 케이블 허브 연결 확인)

![EtherCAT Hub](/images/ethercat_hub.png?width=300px)


- [ ] UPC 랜선 포트들 중 좌측의 EtherCAT 드라이버 패치된 포트에 EtherCAT 허브에서 나온 선을 연결할 것
- [ ] 핸드용 별도의 전원 인가 확인 (12V)
