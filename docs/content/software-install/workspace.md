+++
title = "워크스페이스 구성 및 빌드"
weight = 2
+++

시스템은 두 호스트에서 ROS 2 워크스페이스를 운영합니다.

- **RBY1 UPC** (`nvidia@192.168.2.21`, 워크스페이스 `~/Workspace/aidin_ws`): 실기 하드웨어 인터페이스와 컨트롤러를 구동하는 로봇측 호스트
- **Teleop PC** (`cobot-ai@192.168.2.31`): 입력 디바이스 드라이버·텔레오퍼레이션·MCAP 녹화를 구동하는 작업자측 호스트
  - `~/Workspace/lgit_ws`: 텔레오퍼레이션 / 데이터 수집 워크스페이스
  - `~/Workspace/e2e_ws`: 모방학습 학습 · 추론 워크스페이스

## RBY1 UPC

### 디렉토리 구조

```
~/Workspace/aidin_ws/
└── src/
    ├── aidin_rby1_description
    ├── aidin_rby1_hardware
    ├── aidin_rby1_controller
    ├── aidin_hand_description
    ├── aidin_hand_hardware
    └── aidin_hand_controllers
```

### 소스 가져오기

```bash
mkdir -p ~/Workspace/aidin_ws/src && cd ~/Workspace/aidin_ws/src
git clone https://github.com/aidin-lgit/aidin_rby1_description.git
git clone https://github.com/aidin-lgit/aidin_rby1_hardware.git
git clone https://github.com/aidin-lgit/aidin_rby1_controller.git
git clone https://github.com/aidin-lgit/aidin_hand_description.git
git clone https://github.com/aidin-lgit/aidin_hand_hardware.git
git clone https://github.com/aidin-lgit/aidin_hand_controllers.git
```

### 의존성 해소 · 빌드

```bash
cd ~/Workspace/aidin_ws
colcon build --symlink-install
source install/setup.bash
```

### 빌드 검증

```bash
ros2 pkg list | grep aidin
ros2 launch aidin_rby1_description description.launch.py   # URDF 확인
```

## Teleop PC

### 디렉토리 구조 — `lgit_ws` (텔레오퍼레이션 / 데이터 수집)

```
~/Workspace/lgit_ws/
└── src/
    └── aidin_rby1_teleop/        # 메타패키지 (하위는 git submodule)
        ├── aidin_rby1_description    # RViz 시각화 · 좌표계 참조용
        ├── aidin_rby1_teleop_bringup # 통합 launch 진입점 (robot_launch.sh, teleop_bringup.launch.py 등)
        ├── aidin_rby1_vr_teleop      # Meta Quest VR HMD 입력 노드
        ├── aidin_rby1_vive_teleop    # VIVE Tracker 입력 + 키보드/페달 상태머신
        ├── aidin_manus               # MANUS Glove 손가락 입력 노드
        └── ros2_mcap_recorder        # MCAP 녹화 · HDF5 변환 도구
```

### 소스 가져오기

```bash
mkdir -p ~/Workspace/lgit_ws/src && cd ~/Workspace/lgit_ws/src
git clone --recursive https://github.com/aidin-lgit/aidin_rby1_teleop.git
```

### 의존성 해소 · 빌드

```bash
cd ~/Workspace/lgit_ws
colcon build --symlink-install
source install/setup.bash
```

### 빌드 검증

```bash
ros2 pkg list | grep aidin
ros2 topic list   # UPC 측 컨트롤러 토픽이 보이는지 확인 (ROS_DOMAIN_ID 동일 필요)
```

### 디렉토리 구조 — `e2e_ws` (모방학습 학습 · 추론)

```
~/Workspace/e2e_ws/
└── src/
    ├── e2e_training              # 모방학습 학습 파이프라인 (e2e-train)
    └── e2e_inference             # 학습 모델 추론 노드 (e2e-infer)
```

### 소스 가져오기

```bash
mkdir -p ~/Workspace/e2e_ws/src && cd ~/Workspace/e2e_ws/src
git clone https://github.com/aidin-lgit/e2e_training.git
git clone https://github.com/aidin-lgit/e2e_inference.git
```

### 의존 라이브러리 설치

`e2e_training` 과 `e2e_inference` 는 동일 Python 가상환경을 공유하며, 가상환경 생성·의존성 설치·editable 설치(`pip install -e .`) 절차는 각 패키지의 설치 가이드를 참고하세요.

- [`e2e_training`](https://github.com/aidin-lgit/e2e_training) — README 및 `docs/quickstart.md`
- [`e2e_inference`](https://github.com/aidin-lgit/e2e_inference) — README 및 `docs/user_guide.md`
