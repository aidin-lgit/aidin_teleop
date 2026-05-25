+++
title = "의존성 설치"
weight = 1
+++

## 시스템 패키지

```bash
sudo apt update
sudo apt install -y \
    build-essential cmake git python3-pip \
    libeigen3-dev libopencv-dev \
    ros-humble-desktop-full \
    ros-humble-ros2-control ros-humble-ros2-controllers
```

## NVIDIA / CUDA

- 드라이버: NVIDIA 535+ (Quest 인코딩 NVENC 지원)
- CUDA Toolkit 12.x
- cuDNN: PyTorch 권장 버전 매칭

## Python 패키지

```bash
pip install -r requirements.txt
```

주요 항목:

- `torch`, `torchvision` (CUDA 빌드)
- `numpy`, `scipy`, `h5py`, `zarr`, `pandas`
- `mcap`, `mcap-ros2-support`
- `lerobot` 또는 자체 학습 프레임워크 *TODO*

## SteamVR / MANUS / Quest

- SteamVR (VIVE Tracker 인식용)
- MANUS Core (글로브 캘리브레이션·브릿지)
- Meta Quest Developer Hub / adb

> *TODO: 실제 requirements.txt 경로, 버전 핀, GPU 드라이버 검증 스크립트*
