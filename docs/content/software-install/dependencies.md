+++
title = "의존성 설치"
weight = 1
+++

## ROS2 시스템 패키지

```bash
sudo apt update
sudo apt install -y \
    build-essential cmake git python3-pip \
    libeigen3-dev libopencv-dev \
    ros-humble-desktop-full \
    ros-humble-ros2-control ros-humble-ros2-controllers
```

## NVIDIA / CUDA

- 드라이버: NVIDIA 535+
- CUDA Toolkit 12.x
- cuDNN: PyTorch 권장 버전 매칭

## Python 패키지

```bash
pip install torch torchvision \
    numpy scipy h5py zarr pandas \
    mcap mcap-ros2-support
```

> 위는 핵심 패키지만 나열한 것이며, 추가 의존 라이브러리·버전 핀·CUDA 빌드 옵션 등은 각 패키지의 공식 README / 설치 가이드를 참조하세요.
> - [PyTorch 설치 가이드](https://pytorch.org/get-started/locally/) (CUDA 버전 매칭)
> - [mcap-ros2-support](https://github.com/foxglove/mcap/tree/main/python/mcap-ros2-support)

## SteamVR / MANUS / Quest

- [SteamVR](https://store.steampowered.com/app/250820/SteamVR/) (VIVE Tracker 인식용)
- [MANUS Core](https://www.manus-meta.com/resources/downloads) (글로브 캘리브레이션·브릿지)
- [ZED SDK](https://www.stereolabs.com/developers/release)
