+++
title = "데이터셋 구조"
weight = 4
+++

## 디렉토리 레이아웃

```
/data/
├── raw/                       # MCAP 원본 (불변)
│   ├── pick_red_block_0001_20260601_jm.mcap
│   └── pick_red_block_0001_20260601_jm.yaml
├── hdf5/                      # 변환된 학습용 데이터
│   └── pick_red_block_0001.hdf5
├── index/
│   ├── dataset_index.csv      # 검증 통과한 에피소드 목록
│   └── splits/
│       ├── train.txt
│       ├── val.txt
│       └── test.txt
└── stats/
    └── pick_red_block_norm.npz  # 학습용 정규화 통계
```

## 권장 운영 규칙

- **raw/ 는 read-only 마운트** 또는 별도 백업 디스크
- HDF5 변환은 항상 재현 가능해야 함 (변환 스크립트 + config 버전을 함께 보존)
- `dataset_index.csv` 와 `splits/*.txt` 는 Git LFS 또는 별도 데이터 저장소로 버전 관리
- 정규화 통계 (`stats/*.npz`) 는 학습 시 사용된 split 기준으로만 계산하여 학습-검증 누수 방지

## 태스크별 디렉토리 분리 (대안)

데이터가 늘어나면 다음과 같이 태스크별 디렉토리 분리도 고려.

```
/data/hdf5/
├── pick_red_block/
├── pour_water/
└── fold_towel/
```
