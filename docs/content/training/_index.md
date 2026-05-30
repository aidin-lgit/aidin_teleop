+++
title = "학습"
type = "chapter"
weight = 7
+++

## 개요

텔레옵으로 수집한 HDF5 데이터를 입력으로, **`e2e_training` 툴킷**을 사용해
Diffusion Policy 모방학습 정책을 학습합니다. 학습 결과물(`run_dir`)은 그대로
[추론](../inference/) 단계의 입력이 됩니다.

```
텔레옵 수집 ──▶ HDF5 데이터 ──▶ e2e-train ──▶ run_dir (체크포인트) ──▶ e2e-infer (실로봇 추론)
```

학습 파이프라인은 **CLI(`e2e-train`) + YAML 설정** 으로 구성됩니다. 설정은 YAML 파일에
모아두고, 실행 시점에 `-o key=value` 로 덮어쓰는 방식입니다.

> **이 챕터는 운영 흐름 중심입니다.** 전체 CLI 옵션·설정 키 레퍼런스는 [`e2e_training` README](https://github.com/aidin-lgit/e2e_training)
> 와 docs([`configuration.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/configuration.md), [`data_format.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/data_format.md) 등)를 참고하세요.
> 매뉴얼과 README 의 역할 구분은 [문서 범위](../reference/doc-scope/)를 보세요.

## 이 챕터에서 다루는 내용

- [설치 및 환경 구성](environment/) — venv, 공용 lock 설치, 더미 데이터로 동작 검증
- [데이터 포맷](data-format/) — 공통 HDF5 구조, 관절/TCP 키, state 벡터 레이아웃
- [모델: Diffusion Policy](models/) — 모델 구조, TCP(rot6d) 학습, 정규화
- [학습 실행 · 모니터링 · 평가](training-run/) — `e2e-train` 실행, 다중 GPU(DDP), 체크포인트/resume, TensorBoard, `e2e-eval`

> 현재 `e2e_training` 이 기본 제공하는 모델은 **Diffusion Policy** 단일 모델입니다.
> 새 모델·로봇 추가 워크플로는 [`docs/adding_a_model.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/adding_a_model.md),
> [`docs/adding_a_robot.md`](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/adding_a_robot.md) 를 참고하세요.
