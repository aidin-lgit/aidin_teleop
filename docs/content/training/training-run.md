+++
title = "학습 실행 · 모니터링 · 평가"
weight = 4
+++

## 기본 실행

학습은 `e2e-train` CLI 로 실행합니다. 설정은 `-c` 로 YAML 을 지정하고, 개별 값은
`-o key=value` 로 덮어씁니다.

```bash
e2e-train -c configs/train/diffusion_policy.yaml \
  -o dataset.hdf5_paths=/path/to/your/data.hdf5
```

여러 HDF5 묶기:

```bash
e2e-train -c configs/train/diffusion_policy.yaml \
  -o "dataset.hdf5_paths=[/path/a.hdf5,/path/b.hdf5]"
```

정상 실행 시 학습 진행률과 함께 loss 가 출력됩니다.

```
08:00:01 [INFO] 샘플 수(train): 9800  배치 수: 612  state_dim: 48
학습 진행:  10%|██        | 100/1000  loss=0.0842  lr=1.00e-04
```

- **loss**: 초기 0.8~1.0 → 수렴 시 0.01~0.05 수준이 정상
- 자주 쓰는 오버라이드: `epochs`, `batch_size`, `device.gpu_ids`, `dataset.target_fps`

> 다중 GPU(DDP)·GPU 메모리 절약(`gradient_accumulation_steps`, `mixed_precision`)·
> LR 스케일링·전체 옵션은 [`e2e_training` README](https://github.com/aidin-lgit/e2e_training) / [`docs/configuration.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/configuration.md) 를 참고하세요.
> 예: `-o "device.gpu_ids=[0,1,2,3]"` 로 DDP 학습 (`torchrun` 불필요).

## 체크포인트

```
artifacts/train/diffusion_policy/<run_dir>/
  ckpt/
    best.pt        ← val_loss(있을 때) 또는 train_loss 최저 시점 ← 추론에 사용
    epoch_*.pt     ← checkpoint_interval 마다 저장
    last.pt        ← 최신 주기 체크포인트
  config.yaml      ← 학습 설정 스냅샷
  stats.json       ← 정규화 통계
```

> 추론 단계에서 `run_dir` 의 `config.yaml`, `stats.json`, `ckpt/best.pt` **세 파일이
> 모두 필요**합니다. → [추론 설치 및 설정](../../inference/model-deploy/)

학습이 중단됐다면 `run_dir` 만 지정해 이어서 돌릴 수 있습니다.

```bash
e2e-train --resume <run_dir>             # config/stats/ckpt 자동 사용
e2e-train --resume <run_dir> -o epochs=1500
```

> `joint_keys`·`image_keys` 등 모델 구조 파라미터는 resume 시 바꾸면 로드에 실패합니다.
> resume 은 `epochs`·`lr` 같은 학습 일정 변경에만 안전합니다.

## 모니터링

```bash
tensorboard --logdir artifacts/train/diffusion_policy/tensorboard
```

`train/loss_*`, `val/loss_epoch`, lr, 그리고 매 epoch saliency heatmap 이 기록됩니다.
`dataset.val_ratio > 0` 이면 매 epoch val_loss 를 계산하고 `best.pt` 를 그 기준으로 갱신합니다.

## 오프라인 평가

```bash
e2e-eval -c configs/eval/offline.yaml \
  -o checkpoint_path=<run_dir>/ckpt/best.pt \
  -o dataset_path=/path/to/test_data.hdf5
```

결과는 `<run_dir>/eval/` 에 JSON 지표·PNG 플롯으로 저장됩니다.
실로봇 성능 평가는 [추론 → 출력 검증 및 문제 해결](../../inference/safety-evaluation/) 을 참고하세요.
