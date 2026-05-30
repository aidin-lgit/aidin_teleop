+++
title = "FAQ"
weight = 2
+++

## Q. 트래커 캘리브레이션은 왜 필요 없나요?

본 시스템은 허리 트래커를 RBY1 `base_footprint` 와 고정 변환으로 연결하여, SteamVR 절대 좌표계에 의존하지 않습니다.

## Q. 로봇손(AIDIN Hand)을 다른 핸드로 변경할 수 있나요?

가능합니다. 본 시스템은 `ros2_control` 의 표준 `hardware_interface` 추상화 위에 구축되어 있어, 동일한 컨트롤러·텔레옵 스택을 유지한 채 **로봇손만 다른 그리퍼·핸드로 교체**할 수 있습니다.

교체는 다음 두 단계로 이루어집니다.

1. 교체할 핸드용 `SystemInterface` (또는 `ActuatorInterface`) 플러그인을 [`ros2_control` 의 hardware_interface 규격](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/writing_new_hardware_component.html) 에 맞춰 제작합니다.
2. URDF 의 `<ros2_control>` 블록에서 새 플러그인을 로드하도록 변경하고, command/state interface 이름을 기존 컨트롤러가 기대하는 인터페이스 이름과 매핑합니다.

인터페이스 이름을 동일하게 유지하면 상위 `aidin_hand_controllers` / `aidin_rby1_controller` / 텔레옵 스택은 수정 없이 그대로 동작합니다. 자세한 작성법과 라이프사이클 규약은 [ros2_control 공식 문서](https://control.ros.org/rolling/index.html) 를 참고하세요.

> 단, 손의 관절 구성(DOF)이 달라지면 학습된 정책의 입력·출력 차원도 바뀝니다. 새 핸드로
> 데이터를 다시 수집해 재학습하고, 추론 시 joint name map 을 새 관절 이름으로 갱신해야
> 합니다. → [추론 → 설치 및 설정 (다른 그리퍼/핸드 교체 시)](../../inference/model-deploy/#다른-그리퍼핸드로-교체-시)

## Q. 카메라 변경이 가능한가요?

가능합니다. 카메라는 `ros2_control` 로 통합되어 있지 않고 일반 ROS 2 토픽으로 데이터가 흐르므로, 하드웨어 인터페이스를 새로 만들 필요가 없습니다. 다음 두 가지만 맞추면 됩니다.

1. 새 카메라가 발행하는 **이미지 토픽 이름**을 [`ros2_mcap_recorder/config/topics.yaml`](../../data-collection/format/#기록-대상-토픽) 의 녹화 화이트리스트에 추가합니다.
2. **추론(`e2e_inference`) 단계** 에서 사용하는 카메라 토픽 설정을 새 토픽 이름과 일치시킵니다 (학습 시 사용한 토픽 이름과 동일하게 유지).

토픽 이름만 일관되게 맞추면 카메라 제조사·드라이버 (Realsense / ZED / 일반 USB 등) 와 무관하게 동일한 학습·추론 파이프라인이 동작합니다.

## Q. 학습 모델을 변경할 수 있나요?

가능합니다. 본 시스템은 **데이터와 모델을 분리**하도록 설계되어, 동일한 공통 HDF5 데이터셋 위에서 학습 모델만 교체할 수 있습니다.

- 기본 제공 모델은 **Diffusion Policy** 한 가지입니다 (`configs/train/diffusion_policy.yaml`).
- 새 모방학습 모델을 추가하려면 `e2e_training` 의 레지스트리/템플릿 기반 워크플로를 따릅니다. 모델 클래스를 등록하고 `configs/train/<model>.yaml` 설정을 추가하면, 기존에 수집한 HDF5 데이터와 추론 파이프라인은 그대로 재사용됩니다.
- 자세한 절차는 [`adding_a_model.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/adding_a_model.md) 를 참고하세요.

## Q. 시뮬레이션에서 먼저 학습할 수 있나요?

현재 본 매뉴얼에는 포함되어 있지 않습니다. Isaac Sim 등 시뮬레이션 환경 통합은 추후 업데이트 예정입니다.
