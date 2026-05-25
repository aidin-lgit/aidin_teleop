+++
title = "End-to-End: Pick & Place"
weight = 1
+++

> 본 튜토리얼은 빨간 블록을 잡아 트레이로 옮기는 단순 작업을 통해 전체 파이프라인을 한 번 돌려보는 것이 목적입니다.

## 0. 준비물

- RBY1 + AIDIN Hand, VIVE × 3, MANUS, Meta Quest (셋업 완료 상태)
- 빨간 블록 1개, 흰 트레이 1개
- 학습 PC (별도 또는 동일 PC)

## 1. 텔레옵으로 30 에피소드 수집

```bash
# 매 에피소드마다 index 만 바꾸어 수집
ros2 launch aidin_rby1_teleop record.launch.py \
    episode_name:=pick_red_block_$(printf %04d $i) \
    operator:=jm
```

- 블록 위치를 작업 영역 안에서 다양하게 변경
- 성공 에피소드만 라벨링 (`success: true`)

## 2. MCAP → HDF5 변환

```bash
for f in /data/raw/pick_red_block_*.mcap; do
  ros2 run aidin_rby1_teleop mcap_to_hdf5 \
      --input  "$f" \
      --output "/data/hdf5/$(basename "${f%.mcap}").hdf5" \
      --config aidin_rby1_teleop/config/mcap_to_hdf5.yaml
done
```

## 3. 검증 및 인덱스 생성

```bash
ros2 run aidin_rby1_teleop validate_episode --hdf5 /data/hdf5/*.hdf5
python tools/build_index.py /data/hdf5/ --task pick_red_block --out /data/index/dataset_index.csv
python tools/make_splits.py --index /data/index/dataset_index.csv --train 0.8 --val 0.1 --test 0.1
```

## 4. Diffusion Policy 학습

```bash
cd training
python scripts/train.py \
    model=diffusion_policy \
    data=aidin_dual_arm \
    train.epochs=300 \
    train.exp_id=dp_pick_red_block_v1
```

## 5. 체크포인트 패키징

```bash
python scripts/pack_deploy.py \
    --run runs/dp_pick_red_block_v1 \
    --out /data/deploy/pick_red_block_v1
```

## 6. 자율 추론

```bash
ros2 launch aidin_rby1_teleop inference.launch.py \
    deploy_dir:=/data/deploy/pick_red_block_v1
```

GUI 에서 `Engage` 후 블록 위치를 학습 분포 내에서 무작위로 두고 30회 평가.

## 7. 평가 기록

성공률·사이클 시간을 평가 시트에 기록하고, 부족하면 추가 에피소드 수집 → 재학습.
