+++
title = "모델"
weight = 3
+++

## 기본 지원 모델

### Diffusion Policy (기본값)

- 참고: Chi et al., *Diffusion Policy: Visuomotor Policy Learning via Action Diffusion*
- 본 시스템의 기본 baseline. 이미지 + 상태 → 액션 시퀀스 디노이징
- 설정: `conf/model/diffusion_policy.yaml`

### ACT (선택)

- 참고: Zhao et al., *Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware*
- 양손 작업에 적합. action chunking + transformer encoder/decoder
- 설정: `conf/model/act.yaml`

### BC-RNN / 기타

- 빠른 베이스라인 비교용
- 설정: `conf/model/bc_rnn.yaml`

## 신규 모델 통합

새 모델을 추가하려면 [Data Loader 가이드의 "신규 모델 추가 절차"](../data-loader/#신규-모델-추가-절차) 참고.

## 모델별 입출력 요구

| 모델 | 입력 obs | 출력 action | 비고 |
| --- | --- | --- | --- |
| Diffusion Policy | 2 step obs + image | 16 step action | UNet1D / Transformer |
| ACT | 1 step obs + image | 100 step action | chunking |
| BC-RNN | 1 step obs (+image) | 1 step action | recurrent |

## 양손/그리퍼 차원

| 모델 | EE pose 표현 | 손 차원 |
| --- | --- | --- |
| 기본 | xyz + quat (7) × 2 | AIDIN Hand 관절 수 × 2 |
| 회전 표현 대안 | 6D 회전 (Zhou et al., 2019) | 동일 |

> *TODO: 각 모델별 학습 시간·VRAM·성능 벤치마크 표*
