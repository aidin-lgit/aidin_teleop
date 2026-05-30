+++
title = "추론 실행"
weight = 2
+++

## 기본 실행 명령

설치·설정을 마쳤다면([설치 및 설정](../model-deploy/)) `e2e-infer` 로 추론을 실행합니다.

```bash
e2e-infer \
  --run_dir <e2e_training_run_dir> \
  --checkpoint best \
  --robot rb_y1 \
  --configs_dir <e2e_training_경로>/configs \
  --rby1_joint_name_map configs/inference/rby1_joint_names.yaml \
  --runtime_config configs/inference/runtime.yaml \
  --device cuda
# frequency / steps_per_inference / max_duration 은 모두 선택사항.
# 미지정 시 runtime.yaml → 내장 기본값 순으로 적용되며,
# frequency 는 학습 config 의 dataset.fps 를 자동 사용한다.
```

`Ctrl+C` 로 종료합니다.

## 체크포인트 선택 (`--checkpoint`)

`run_dir/ckpt/` 안의 체크포인트를 여러 형태로 지정할 수 있습니다 (기본값 `best`).

| 입력 | 매칭 파일 |
|---|---|
| `best` 또는 `best.pt` | `best.pt` |
| `last` 또는 `last.pt` | `last.pt`(있으면) → 없으면 가장 큰 `epoch_*.pt` |
| `real_best` | `real_best.pt` / `REAL_best.pt` (대소문자 무시) |
| `500` 또는 `epoch_500` | `epoch_0500_loss….pt` |
| `epoch_0500_loss0.0058.pt` | 정확한 파일명 그대로 |

사용 가능한 체크포인트 목록만 보려면:

```bash
e2e-infer --list_checkpoints \
  --run_dir <run_dir> --robot rb_y1 --configs_dir <e2e_training>/configs
```

## 실행 흐름

```
1. 체크포인트 로드 (config.yaml, stats.json, ckpt/<resolved>.pt)
2. ROS2 /joint_states 구독 시작 (백그라운드 스레드)
3. ROS2 카메라 이미지 토픽 구독 시작 (백그라운드 스레드)
4. 에피소드 루프:
   a. obs 수집 (관절 + 카메라, 타임스탬프 정렬)
   b. Policy.predict(obs) → action 시퀀스
   c. steps_per_inference 개 액션을 타임스탬프에 맞춰 실행
      ├─ publish: true  → ROS2 토픽 발행
      └─ publish: false → 콘솔 출력만
   d. max_duration 초 경과 시 에피소드 종료
5. Ctrl+C 입력 시 종료
```

## 주요 파라미터

> 우선순위 규칙: **CLI 인자 > runtime.yaml > 내장 기본값**.

| 옵션 | 기본값 | 설명 |
|---|---|---|
| `--frequency` | 학습 `dataset.fps` | 제어 주파수(Hz). 학습 fps 와 10% 이상 차이 나면 추론을 거부(SystemExit) |
| `--steps_per_inference` | `6` | 한 번 추론(`pred_horizon` 개) 중 실제 실행할 앞쪽 스텝 수. 작을수록 재추론 잦음 → 외란 대응↑, 비용↑ |
| `--max_duration` | `60` | 한 에피소드 최대 시간(초). 첫 구동은 10–20초로 짧게 |
| `--use_ema` | off | EMA 가중치 사용 (일반적으로 더 안정적) |
| `--device` | `cuda` | `cuda` 또는 `cpu` (CPU 는 느리므로 `--frequency` 를 낮추세요) |

> `--steps_per_inference` 는 runtime.yaml 에서 `n_action_steps` 키로 노출됩니다.
> `pred_horizon` 보다 큰 값을 지정하면 시작 시 즉시 거부됩니다.

## TCP 모델 추론 (자동 pose_repr 파이프라인)

체크포인트의 `model.tcp_keys` 가 비어있지 않으면 **자동으로 TCP pose 파이프라인**이 적용됩니다.
별도 옵션은 없습니다 — joint vs TCP 는 학습 config 로만 결정됩니다. 시작 로그로 모드를 확인할 수 있습니다.

```
[policy] mode=tcp_relative  tcp_keys=['pose_eef_L', 'pose_eef_R']  obs_pose_repr=abs  action_pose_repr=relative
[policy] mode=joint  (model.tcp_keys 비어있음)
```

`action_pose_repr=relative` 모델은 상대 액션을 절대 좌표로 복원해 발행하는 과정이 자동으로
처리됩니다. 좌표 변환 파이프라인 상세는 `e2e_inference` [`docs/user_guide.md`](https://github.com/aidin-lgit/e2e_inference/blob/release/v1.1.0/docs/user_guide.md) 를 참고하세요.

## DDIM step 수 조정 (`num_inference_steps`)

학습 ckpt 에는 기본 `num_inference_steps=8` 이 저장돼 있지만, 추론 시 덮어쓸 수 있습니다
(우선순위: CLI > runtime.yaml > 학습 default).

```bash
e2e-infer --num_inference_steps 16 ...
```

| step | 특성 |
|---|---|
| 8  | 빠르지만 sampling 노이즈 큼 — 출력이 흔들릴 수 있음 |
| 16 | 안정성·속도 균형 — **권장 기본값** |
| 32 | 더 안정적이지만 predict_dt 가 ~2× |

> 비동기 모드(`async_inference: true`) + 낮은 step 조합은 jerk 가 누적되므로 16 이상 권장.
> 비동기 추론·컨트롤러 temporal ensemble 등 고급 런타임 옵션은 `e2e_inference`
> [`docs/user_guide.md`](https://github.com/aidin-lgit/e2e_inference/blob/release/v1.1.0/docs/user_guide.md) 를 참고하세요.

## 콘솔 출력을 파일로 저장

`publish: false` 출력값을 분석하려면 stdout 을 파일로 저장하세요.

```bash
mkdir -p outputs
e2e-infer ... 2>&1 | tee outputs/run_$(date +%Y%m%d_%H%M%S).log
```

출력값을 어떻게 검증하는지는 [출력 검증 및 문제 해결](../safety-evaluation/) 을 참고하세요.
