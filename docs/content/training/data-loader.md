+++
title = "다중 모델 호환 Data Loader 가이드"
weight = 2
+++

## 설계 목표

서로 다른 모방학습 모델 (Diffusion Policy, ACT, BC-RNN, Octo 등) 이
**동일한 HDF5 데이터셋 위에서** 최소한의 어댑터로 학습될 수 있도록 한다.

핵심 분리:

- **HDF5 스키마**(저장된 원본 구조) ←→ **Sample dict 스키마**(모델 입력 직전 형태)

이 두 계층을 명시적으로 분리하고, 사이를 **매핑(mapping) yaml** 로 연결한다.

## 계층 구조

```
HDF5 episode files
        │
        ▼
EpisodeReader        (HDF5 → episode dict; per-episode metadata 포함)
        │
        ▼
WindowSampler        (시퀀스 윈도우 자르기: obs_horizon, action_horizon)
        │
        ▼
FieldMapper          (yaml 매핑 적용: 채널 선택/이름 변경/정규화/이미지 전처리)
        │
        ▼
ModelAdapter         (모델별 dict 키 규약에 맞게 마지막으로 재배열)
        │
        ▼
torch.utils.data.Dataset
```

## 매핑 yaml 예시

```yaml
# conf/data/aidin_dual_arm.yaml
obs:
  joint_pos:       /obs/joint_pos
  hand_left:       /obs/hand_left_joint_pos
  hand_right:      /obs/hand_right_joint_pos
  wrist_left:      /obs/wrist_left_pose
  wrist_right:     /obs/wrist_right_pose
  image_left:      /obs/image_left
  image_right:     /obs/image_right
action:
  wrist_left_tgt:  /action/wrist_left_target
  wrist_right_tgt: /action/wrist_right_target
  hand_left_tgt:   /action/hand_left_target
  hand_right_tgt:  /action/hand_right_target

normalize:
  joint_pos:  {type: zscore, stats: stats/joint_pos.npz}
  wrist_left: {type: pose, position: zscore, rotation: passthrough}

image:
  resize: [224, 224]
  to_float: true
  augment:
    color_jitter: 0.1
    random_crop: 0.05

window:
  obs_horizon: 2
  action_horizon: 16
  stride: 1
```

## 모델별 어댑터 규약

| 모델 | obs 키 | action 키 | shape 규약 |
| --- | --- | --- | --- |
| Diffusion Policy | `obs.{name}: (B, T_obs, D)` | `action: (B, T_act, D_act)` | flattened action |
| ACT | `obs.{name}: (B, T_obs, D)` | `action: (B, T_act, D_act)` | qpos + ee + gripper |
| BC-RNN | `obs.{name}: (B, 1, D)` | `action: (B, 1, D_act)` | single step |
| LeRobot 어댑터 | LeRobot 의 `observation.state`, `action` 규약을 따름 | | |

어댑터는 모두 `ModelAdapter` 추상 클래스를 구현하며, 매핑 yaml + 모델 이름만으로 학습 스크립트가 자동 선택한다.

## 신규 모델 추가 절차

1. `training/data_loader/adapters/<model_name>.py` 에 `ModelAdapter` 서브클래스 작성
2. `conf/model/<model_name>.yaml` 에 모델 설정과 사용할 obs 키 목록 명시
3. 매핑 yaml 은 데이터셋이 동일하다면 재사용

> *TODO: 실제 ModelAdapter 인터페이스 코드 링크, 단위 테스트 작성 가이드*
