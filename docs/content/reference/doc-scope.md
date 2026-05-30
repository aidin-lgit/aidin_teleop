+++
title = "문서 범위: 매뉴얼 vs README"
weight = 1
+++

이 문서 사이트와 각 패키지의 GitHub README 는 **역할이 다릅니다.** 무엇을 어디서 찾아야
하는지, 그리고 문서를 작성·수정할 때 어느 쪽에 써야 하는지에 대한 기준입니다.

## 경계 원칙

| | **이 매뉴얼 (문서 사이트)** | **패키지 README (GitHub)** |
|---|---|---|
| 한 줄 정의 | **"어떻게 운영하는가"** | **"어떻게 빌드·개발·확장하는가"** |
| 대상 독자 | operator / 사용자 | 개발자 / 시스템 관리자 |
| 다루는 것 | 이미 설치된 시스템을 켜고, 텔레옵·데이터 수집·학습·자율 실행까지 가는 **흐름과 절차** | 의존성·빌드, 전체 CLI/설정 레퍼런스, 코드 구조, 새 모델·로봇·하드웨어 추가 |
| 예시 | "체크포인트로 자율 실행하는 절차", "E-Stop 복구", "데이터 수집 체크리스트" | `e2e-train` 전체 옵션표, config 키 레퍼런스, DDP 내부, 모델 추가 워크플로 |

> 한 페이지가 두 성격을 모두 담지 않도록 합니다. 깊은 레퍼런스가 필요하면 **매뉴얼에는
> 운영 흐름만 두고, 상세는 README 로 링크**합니다.

## "README 로 링크" 컨벤션

매뉴얼에서 개발자 레벨 상세가 필요한 지점에는 다음 형식의 안내를 답니다.

```markdown
> 전체 옵션·설정 레퍼런스는 `<패키지>` README 를 참고하세요.
> 이 페이지는 운영 흐름만 다룹니다.
```

## 패키지 ↔ 문서 매핑

| 주제 | 매뉴얼 위치 | 상세 README |
|---|---|---|
| 의존성·빌드 | [소프트웨어 설치](../../software-install/) (요구사항·검증만) | 각 패키지 README |
| 텔레옵 상태머신·런치 인자 | [원격제어](../../teleoperation/) | [`aidin_rby1_teleop`](https://github.com/aidin-lgit/aidin_rby1_teleop) · [`aidin_rby1_teleop_bringup`](https://github.com/aidin-lgit/aidin_rby1_teleop_bringup) · [`aidin_rby1_vive_teleop`](https://github.com/aidin-lgit/aidin_rby1_vive_teleop) |
| MCAP 기록·토픽/HDF5 스키마 | [데이터 수집](../../data-collection/) (절차) | [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder), [`e2e_training` data_format](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/data_format.md) |
| 학습 CLI·config 레퍼런스 | [학습](../../training/) (워크플로) | [`e2e_training` README](https://github.com/aidin-lgit/e2e_training) + [docs/configuration.md](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/configuration.md) |
| 추론 CLI·런타임 설정 | [추론](../../inference/) (워크플로) | [`e2e_inference` README](https://github.com/aidin-lgit/e2e_inference) + [docs/user_guide.md](https://github.com/aidin-lgit/e2e_inference/blob/release/v1.1.0/docs/user_guide.md) |
| 새 모델·로봇·하드웨어 추가 | — (매뉴얼 비포함) | [adding_a_model.md](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/adding_a_model.md), [adding_a_robot.md](https://github.com/aidin-lgit/e2e_training/blob/release/v1.0.1/docs/adding_a_robot.md) |
