+++
title = "RBY1 + AIDIN Hand 조립"
weight = 1
+++

## 조립 순서

1. RBY1 양 손목 플랜지에서 기본 그리퍼 제거 — [Rainbow Robotics 공식 가이드: Detaching Guide](https://rainbowrobotics.github.io/rby1-dev/maintenance/detaching_guide_prev.html) 절차를 따를 것
2. AIDIN Hand 부착 — 좌/우 손목 플랜지에 각각 좌/우 핸드 체결

   RBY1 의 ISO 9409-1-50-4-M6 툴 플랜지에 손목 카메라 지그와 커넥터·커넥터 링으로 AIDIN Hand 를 체결합니다.

   ![핸드 체결](/images/hand_connection.png?width=400px)

3. AIDIN Hand 통신 및 전원 케이블 체결
4. 좌/우 통신 케이블을 EtherCAT 허브에 연결
5. 통신·전원 케이블 라우팅 (어깨→팔뚝→손목)

## 체크리스트

- [ ] 좌/우 손 페어 식별 (좌우 케이블 허브 연결 확인)

![EtherCAT Hub](/images/ethercat_hub.png?width=300px)


- [ ] UPC 랜선 포트들 중 좌측의 EtherCAT 드라이버 패치된 포트에 EtherCAT 허브에서 나온 선을 연결할 것

{{% notice style="warning" title="UPC EtherCAT 포트 위치 — 반드시 확인" %}}
RBY1 UPC 의 일반 LAN 포트와 EtherCAT 드라이버 패치된 포트는 외관이 동일하므로 혼동하기 쉽습니다. **반드시 아래 사진의 좌측 포트** 에 EtherCAT 허브 케이블을 연결하세요. 잘못된 포트에 연결하면 EtherCAT 통신이 잡히지 않아 AIDIN Hand 가 동작하지 않습니다.

![UPC EtherCAT 포트 위치](/images/rby1_port.png?width=400px)
{{% /notice %}}

- [ ] 핸드용 별도의 전원 인가 확인 (12V)
