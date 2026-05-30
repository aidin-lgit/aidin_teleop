+++
title = "설치 및 환경 구성"
weight = 1
+++

## 요구 사양

| 항목 | 요구 | 비고 |
|---|---|---|
| Python | 3.10 이상 | |
| PyTorch | 2.0 이상 | CUDA 12.x 빌드 권장 |
| GPU | CUDA 지원 GPU | CPU 도 동작하지만 매우 느림. 학습은 RTX 4090(24 GB+) 급 권장 |

> 학습 PC 는 텔레옵 PC 와 물리적으로 분리하는 것을 권장합니다.

## 설치

`e2e_training` 과 `e2e_inference` 는 **동일한 venv 를 공유**하며, `requirements.txt` 가
**두 패키지 공용 lock 파일** 입니다. 한 번에 의존성을 설치한 뒤 각 레포에서
`pip install -e .` 만 추가로 실행하면 학습/추론을 모두 사용할 수 있습니다.

> 아래는 빠른 시작용 요약입니다. **버전 호환·플랫폼별 상세 설치는 [`e2e_training` README](https://github.com/aidin-lgit/e2e_training)**
> 를 정본으로 따르세요.

```bash
git clone <repo-url>
cd e2e_training

# 가상환경 생성 + 활성화 (시스템 Python 오염 방지)
python3 -m venv env
source env/bin/activate

# 공용 lock 으로 의존성 설치
pip install --upgrade pip
pip install -r requirements.txt \
  --extra-index-url https://download.pytorch.org/whl/cu128

# e2e_training editable 설치
pip install -e .

# 선택: 테스트·린트 도구 포함
pip install -e ".[dev]"
```

> 새 터미널을 열 때마다 `source env/bin/activate` 로 가상환경을 다시 활성화해야
> `e2e-train` 명령이 보입니다. 빠져나올 때는 `deactivate`.

## 더미 데이터로 동작 검증

레포에 동작 검증용 **더미 데이터**(`data_dummy/dataset.hdf5`, ~14 MB)가 포함되어 있어,
별도 데이터셋 없이도 학습 파이프라인을 한 번 돌려볼 수 있습니다. (5 데모 × 60 프레임,
60×80 다운샘플 이미지, 키 구조는 실제 데이터와 동일)

```bash
e2e-train -c configs/train/diffusion_policy.yaml \
  -o dataset.hdf5_paths=data_dummy/dataset.hdf5 \
  -o epochs=10 -o batch_size=4 -o "device.gpu_ids=[0]"
```

수십 초 안에 학습이 시작되어 loss 가 떨어지는 것이 보이면 설치·환경이 정상입니다.

## 디렉토리 구조

```
e2e_training/
├── configs/
│   ├── train/diffusion_policy.yaml   # 학습 설정 (모델/데이터/학습 파라미터)
│   ├── eval/offline.yaml             # 오프라인 평가 설정
│   ├── robots/rb_y1.yaml             # 로봇 프로파일 (팔 7+7, 머리 2, 토르소 6)
│   └── hands/aidin_gen1.yaml         # 핸드 프로파일 (15 DOF/hand)
├── data_dummy/dataset.hdf5           # 동작 검증용 더미 데이터
└── artifacts/train/<model>/<run_dir> # 학습 결과물 (체크포인트/로그/통계)
```

## 데이터 준비

학습 입력은 **공통 HDF5 포맷** 입니다. 구조·키 규약은 [데이터 포맷](../data-format/) 참고.
학습 PC 에서 데이터 경로는 텔레옵 PC 의 원본과 동일한 내용으로 접근 가능해야 합니다
(NFS / rsync / 동일 디스크).
