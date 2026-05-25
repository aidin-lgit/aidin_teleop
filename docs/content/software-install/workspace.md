+++
title = "워크스페이스 구성 및 빌드"
weight = 2
+++

## 디렉토리 구조

```
~/aidin_ws/
└── src/
    ├── aidin_rby1_description
    ├── aidin_rby1_hardware
    ├── aidin_rby1_controller
    ├── aidin_hand_description
    ├── aidin_hand_hardware
    ├── aidin_hand_controllers
    └── aidin_rby1_teleop
```

## 소스 가져오기

```bash
mkdir -p ~/aidin_ws/src && cd ~/aidin_ws/src
git clone https://github.com/aidin-lgit/aidin_rby1_description.git
git clone https://github.com/aidin-lgit/aidin_rby1_hardware.git
git clone https://github.com/aidin-lgit/aidin_rby1_controller.git
git clone https://github.com/aidin-lgit/aidin_hand_description.git
git clone https://github.com/aidin-lgit/aidin_hand_hardware.git
git clone https://github.com/aidin-lgit/aidin_hand_controllers.git
git clone https://github.com/aidin-lgit/aidin_rby1_teleop.git
```

## 의존성 해소

```bash
cd ~/aidin_ws
rosdep install --from-paths src --ignore-src -r -y
```

## 빌드

```bash
colcon build --symlink-install
source install/setup.bash
```

## 빌드 검증

```bash
ros2 pkg list | grep aidin
ros2 launch aidin_rby1_description display.launch.py   # URDF 확인
```

> *TODO: 패키지별 빌드 옵션 / 디버그 빌드 / clean 빌드 가이드*
