+++
title = "데이터 포맷"
weight = 2
+++

## 공통 HDF5 구조

모든 학습은 동일한 **RoboMimic 스타일 HDF5** 를 입력으로 사용합니다 (mcap 으로부터 변환).
`data/<demo>/obs/` 하위에 관절·포즈·이미지 키가 위치합니다.

```
dataset.hdf5
├── data/
│   ├── demo_0/
│   │   └── obs/
│   │       ├── joint_arm_L         (T, 7)        float32, rad
│   │       ├── joint_arm_R         (T, 7)        float32, rad
│   │       ├── joint_hand_L        (T, 15)       float32, rad
│   │       ├── joint_hand_R        (T, 15)       float32, rad
│   │       ├── joint_head          (T, 2)        float32, rad     (선택)
│   │       ├── joint_torso         (T, 6)        float32, rad     (선택)
│   │       ├── pose_eef_L          (T, 7)        float32, m+quat  (선택, TCP 학습 시)
│   │       ├── pose_eef_R          (T, 7)        float32, m+quat  (선택, TCP 학습 시)
│   │       ├── image_head_color    (T, H, W, 3)  uint8, RGB
│   │       ├── image_wrist_L_color (T, H, W, 3)  uint8, RGB       (선택)
│   │       └── image_wrist_R_color (T, H, W, 3)  uint8, RGB       (선택)
│   ├── demo_1/ ...
│   └── ...
└── mask/   # RoboMimic 호환 그룹 (있어도 무방). 학습 코드는 미사용 —
            #   train/val 분할은 dataset.val_ratio 로 episode 단위 랜덤 분할
```

- `T`: 에피소드 길이(프레임 수)
- 각도 단위: **라디안(rad)**
- TCP pose: xyz(m) + quaternion(xyzw, scalar-last) — dataset 단계에서 9D rot6d 로 변환됨
- 이미지: **uint8, RGB**(rgb8). dataloader 는 색공간 변환 없이 그대로 입력 — 모델 입력은 항상 RGB(0~255)

## 관절 키와 state 벡터 구성

`joint_keys` 에 나열된 순서대로 관절 값이 **이어 붙여져** 하나의 state 벡터를 구성합니다.
`tcp_keys` 가 비어있지 않으면 내부 state 레이아웃은 **`tcp_keys + joint_keys`** 순서로 결합됩니다.

config 기본 설정(`configs/train/diffusion_policy.yaml`, TCP 학습 모드):

```yaml
joint_keys: [joint_hand_L, joint_hand_R]
tcp_keys:   [pose_eef_L, pose_eef_R]       # 9D (xyz+rot6d) 로 자동 변환
```

이 경우 state 벡터 구성:

| 인덱스 | 키 | 차원 | 설명 |
|---|---|---|---|
| 0 ~ 8 | `pose_eef_L` | 9 | 왼손 TCP (xyz + rot6d) |
| 9 ~ 17 | `pose_eef_R` | 9 | 오른손 TCP (xyz + rot6d) |
| 18 ~ 32 | `joint_hand_L` | 15 | 왼손 관절 |
| 33 ~ 47 | `joint_hand_R` | 15 | 오른손 관절 |
| **합계** | | **48** | `state_dim` |

### joint-only 변형 (TCP 미사용)

TCP 없이 관절만으로도 학습할 수 있습니다 (`tcp_keys: []`, 팔/머리/허리를 `joint_keys` 에 추가).
이 경우 state 는 `joint_keys` 순서대로만 구성됩니다 (예: 팔 7+7 + 손 15+15 = 44).
키별 차원·조합 규칙의 전체 표는 `e2e_training` [`docs/data_format.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/data_format.md) 를 참고하세요.

> **주의:** 모델 출력의 관절 순서는 학습 시 사용한 `joint_keys`(+ `tcp_keys`) 순서와 동일합니다.
> 이 순서는 [추론](../../inference/) 단계의 joint name map 작성 순서와 정확히 일치해야 합니다.

## 이미지 키

| 키 | 권장 해상도 | 설명 |
|---|---|---|
| `image_head_color`    | 480×640 | 헤드 카메라 (RGB) |
| `image_wrist_L_color` | 480×640 | 왼손목 카메라 (RGB, 선택) |
| `image_wrist_R_color` | 480×640 | 오른손목 카메라 (RGB, 선택) |

config 기본값 `image_keys: [image_head_color, image_wrist_L_color, image_wrist_R_color]` 는
RB-Y1 의 헤드 + 양 손목 카메라 구성입니다. 사용 카메라가 다르면 `image_keys` 를 조정합니다.

> **주의:** `image_keys` 를 바꾸면 모델 구조가 달라져 기존 체크포인트와 호환되지 않습니다.

> 파일마다 해상도가 다르거나(예: 240×320 / 480×640 혼재) 메모리를 줄여야 하면
> `dataset.image_resize: [H, W]` 로 사전 다운샘플링할 수 있습니다. 상세 옵션은
> `e2e_training` [`docs/configuration.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/configuration.md) 를 참고하세요.

## 로봇·핸드 프로파일

관절 DOF 스펙은 코드에 하드코딩하지 않고 `configs/robots/`, `configs/hands/` 에서 관리합니다.
cfg 에서 `dataset.joint_keys` 를 명시하지 않으면 `robot` YAML 의 `joint_keys` 를 자동으로 읽어 옵니다.

```
configs/
  robots/rb_y1.yaml       # 팔 7+7, 머리 2, 토르소 6
  hands/aidin_gen1.yaml   # 15 DOF/hand
```
