+++
title = "학습 실행 및 모니터링"
weight = 4
+++

## 단일 GPU 학습

```bash
cd training
python scripts/train.py \
    model=diffusion_policy \
    data=aidin_dual_arm \
    train.epochs=300 \
    train.batch_size=64 \
    train.lr=1e-4
```

## 멀티 GPU (torchrun)

```bash
torchrun --nproc_per_node=2 scripts/train.py \
    model=diffusion_policy \
    data=aidin_dual_arm \
    train.batch_size=128
```

## 체크포인트 / 로그

- 체크포인트: `runs/<exp_id>/checkpoints/epoch_*.ckpt`
- TensorBoard: `runs/<exp_id>/tb/`
- W&B: `wandb.project=aidin-teleop` 환경에서 자동 동기화

## 권장 모니터링 항목

- `loss/total`, `loss/action`
- `metric/val_mse_action`
- `metric/val_success_rollout` (시뮬레이션 또는 작은 실로봇 평가)
- 입력 분포 변화 (정규화 통계 누락 확인)

## 학습 종료 조건

- 검증 MSE 가 N epoch 이상 감소하지 않으면 early stop
- 학습 중 disengage 마스킹된 구간이 손실에 기여하지 않는지 정기 점검

## 재현성

- Hydra 가 자동으로 `runs/<exp_id>/.hydra/` 에 설정 스냅샷 저장
- 학습에 사용된 `dataset_index.csv`, `splits/*.txt` 의 git hash 도 함께 기록 권장
