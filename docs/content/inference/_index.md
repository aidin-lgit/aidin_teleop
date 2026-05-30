+++
title = "추론"
type = "chapter"
weight = 8
+++

## 개요

`e2e_training` 에서 학습한 체크포인트(`run_dir`)를 로드해, **`e2e_inference` 프레임워크**로
RB-Y1 실로봇에서 추론/롤아웃을 실행합니다.

```
e2e_training (학습) ──▶ run_dir (체크포인트) ──▶ e2e_inference (실로봇 추론)
```

`e2e-infer` 는 ROS2 `/joint_states` 에서 관절 상태를, 카메라 `sensor_msgs/Image` 토픽에서
이미지를 받아 모델 추론을 수행하고, 예측 액션을
`/aidin_rby1_joint_controller/joint_state_command` 로 발행합니다.

> **이 챕터는 운영 흐름 중심입니다.** 전체 CLI 옵션·런타임 설정 레퍼런스는 [`e2e_inference` README](https://github.com/aidin-lgit/e2e_inference)
> 와 [`docs/user_guide.md`](https://github.com/aidin-lgit/e2e_inference/blob/release/v1.1.0/docs/user_guide.md) 를 참고하세요. 매뉴얼/README 역할 구분은
> [문서 범위](../reference/doc-scope/)를 보세요.

> ⚠️ **안전 수칙**
> - 새 체크포인트는 **항상 `publish: false`(안전 모드)로 먼저 검증**한 뒤 실제 구동에 사용하세요.
> - 실제 구동 시 비상정지 버튼이 손에 닿는 위치에 있어야 합니다.
> - 첫 구동 시 `--max_duration` 을 짧게(예: 10초) 설정해 점진적으로 검증하세요.

## 이 챕터에서 다루는 내용

- [설치 및 설정](model-deploy/) — 패키지 설치, joint name map, runtime config(publish 토글)
- [추론 실행](autonomous-task/) — `e2e-infer` 실행, 체크포인트 선택, 실행 흐름, TCP/주요 파라미터
- [출력 검증 및 문제 해결](safety-evaluation/) — `publish:false` 출력 해석, FAQ, 트러블슈팅
