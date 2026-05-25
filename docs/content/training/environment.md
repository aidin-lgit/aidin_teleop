+++
title = "학습 환경 구성"
weight = 1
+++

## 권장 사양

- **GPU**: NVIDIA RTX 4090 이상 (VRAM 24 GB+)
- **CUDA**: 12.x
- **PyTorch**: 2.x
- 학습 PC 는 텔레옵 PC 와 물리적으로 분리 권장 (텔레옵 중 GPU 경합 방지)

## Python 환경

```bash
conda create -n aidin_train python=3.10 -y
conda activate aidin_train
pip install -r training/requirements.txt
```

주요 항목:

- `torch`, `torchvision`
- `h5py`, `zarr`
- `diffusers`, `einops`
- `hydra-core`, `omegaconf`
- `wandb` (실험 추적)
- `lerobot` (옵션, 호환 어댑터로 사용 가능)

## 디렉토리 구조

```
training/
├── conf/                   # Hydra 설정
│   ├── model/
│   ├── data/
│   └── train/
├── data_loader/            # 다중 모델 호환 Data Loader
├── models/                 # Diffusion Policy / ACT / 기타
├── runs/                   # 체크포인트 / 로그
└── scripts/
    └── train.py
```

## 데이터 마운트

학습 PC 에서 `/data/hdf5` 가 텔레옵 PC 의 원본과 동일한 내용으로 마운트되어야 함 (NFS / rsync / 동일 디스크).
