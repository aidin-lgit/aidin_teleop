+++
title = "모델: Diffusion Policy"
weight = 3
+++

## 지원 모델

| 모델 | 키 | 특징 |
|---|---|---|
| Diffusion Policy | `diffusion_policy` | 1D UNet + DDIM, 경량, 빠른 학습 |

> 현재 기본 제공 모델은 Diffusion Policy 단일 모델입니다. 새 모델을 추가하려면
> [`docs/adding_a_model.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/adding_a_model.md) 레지스트리/템플릿 워크플로를 참고하세요.

## Diffusion Policy

> Chi et al. 2023, *Diffusion Policy: Visuomotor Policy Learning via Action Diffusion*

### 구조

```
obs (joint + image)
  → ObsEncoder
      ├─ per-camera: resize → crop → /255 → ×2-1 → ResNet18Conv (BN→GN, scratch)
      │              → SpatialSoftmax(num_kp=32) → Linear(64 → img_feature_dim=64)
      └─ joint state (lowdim, identity)
  → global_cond

random noise
  → ConditionalUnet1D (1D UNet, FiLM conditioning)
  ← global_cond, timestep t
  → predicted noise → DDIM 역방향 → clean action
```

### 학습 방식

- **DDIM 단일 인스턴스**(100 train_timesteps, `squaredcos_cap_v2`)가 학습(`add_noise`)과
  추론(`step`) 양쪽을 처리합니다.
- 학습: noisy action 에서 noise(`epsilon`)를 예측. obs 와 action 은 **각자 자기 분포**에서
  `[-1, 1]` 로 정규화(per-key normalizer).
- 추론: DDIM `num_inference_steps`(기본 8, runtime override 가능)으로 random noise → clean action.

### 입출력 윈도우

| 파라미터 | 기본값 | 설명 |
|---|---|---|
| `dataset.obs_horizon` | `2` | 관측 윈도우 길이(프레임) |
| `dataset.pred_horizon` | `16` | 예측 action 시퀀스 길이(프레임) |

## TCP (xyz + rot6d) 학습

엔드이펙터 pose 로 학습하려면 `tcp_keys` 에 TCP pose 키를 명시합니다. HDF5 의 7D
(xyz + quaternion_xyzw) 가 dataset 단계에서 자동으로 9D(xyz + rot6d)로 변환됩니다.

```yaml
dataset:
  joint_keys: [joint_hand_L, joint_hand_R]
  tcp_keys:   [pose_eef_L, pose_eef_R]
  pose_repr:
    obs_pose_repr:    abs        # obs 는 절대 좌표
    action_pose_repr: relative   # action 만 base = obs[-1] 기준 상대 좌표
```

- rot6d 컨벤션: `[R[:,0]; R[:,1]] = [r00, r10, r20, r01, r11, r21]` (Zhou et al. 2019).
- `action_pose_repr=relative` 로 학습한 체크포인트는 추론 시 `base = obs[-1]` 기준 상대 액션을
  반환하며, 추론 측에서 `base @ action` 으로 절대 좌표를 복원합니다 (추론 단계에서 자동 처리).

## 정규화 통계 (stats.json)

첫 실행 시 obs/action 별로 차원별 min/max 를 계산해 `run_dir/stats.json` 에 저장합니다.
**추론 시 학습 때 만든 `stats.json` 을 그대로 사용**해야 정규화 기준이 맞습니다.

| 슬라이스 | 정규화 |
|---|---|
| TCP 의 xyz (3D)   | range → [-1, 1] |
| TCP 의 rot6d (6D) | identity (단위 회전 표현 보존) |
| `joint_*`, `hand_*` 등 | range → [-1, 1] |

## 이미지 인코더 및 Augmentation

각 카메라 이미지는 resize → crop → 정규화를 거쳐 ResNet18 기반 인코더(SpatialSoftmax)로
특징을 뽑은 뒤 상태와 함께 조건으로 들어갑니다. Augmentation(ColorJitter/RandomGrayscale)은
학습 시에만 적용되고 추론 시 자동 비활성화됩니다 — **조명·색감 변동이 큰 환경에서는 켜는 것을 권장**합니다.

```bash
e2e-train -c configs/train/diffusion_policy.yaml \
  -o model_params.color_jitter=true \
  -o model_params.random_grayscale=true
```

> 인코더 파이프라인 상세와 `model_params`(`down_dims`, `resize_shape`/`crop_shape`,
> `img_feature_dim`, `num_inference_steps` 등) 전체 키는 `e2e_training` [`docs/configuration.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/configuration.md)
> 를 참고하세요.
