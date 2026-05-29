+++
title = "후처리 및 검증"
weight = 3
+++

녹화가 끝난 세션 폴더의 MCAP 파일을 학습용 HDF5 로 변환하고, 변환 결과를 확인하는 단계입니다. 변환 도구는 모두 [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) 패키지가 제공합니다.

## 1. MCAP → HDF5 변환

### 1-1. 세션 일괄 변환 (권장)

세션 폴더 안의 모든 에피소드를 한 번에 RoboMimic 호환 HDF5 로 묶습니다.

```bash
# 가장 최신 session_* 자동 선택 → bundle 모드 (단일 HDF5 에 demo_0, demo_1, ... 누적)
ros2 run ros2_mcap_recorder convert_session_to_hdf5

# 명시적 세션 + per-episode 모드 (에피소드마다 별도 HDF5 파일)
ros2 run ros2_mcap_recorder convert_session_to_hdf5 \
  --session ./logs/session_20260524_153000 \
  --mode per-episode

# 샘플링 주파수를 30 Hz 로 override
ros2 run ros2_mcap_recorder convert_session_to_hdf5 --fps 30
```

출력 위치:

- `bundle` (기본): `<session>/dataset.hdf5` 안에 `data/demo_0`, `data/demo_1`, ... 그룹으로 누적
- `per-episode`: `<session>/dataset_episode_1.hdf5`, `dataset_episode_2.hdf5`, ...

`bundle` 모드는 기존 HDF5 를 항상 덮어쓰며(`demo_*` 인덱스 정합성 보장), `per-episode` 모드는 에피소드별로 새 파일을 씁니다.

### 1-2. 단일 MCAP 변환

세션 안의 특정 에피소드 하나만 변환할 때 사용합니다.

```bash
# 명시적 입력/출력
ros2 run ros2_mcap_recorder convert_mcap_to_hdf5 \
  --mcap logs/session_<ts>/episode_1/episode_1_0.mcap \
  --out  logs/session_<ts>/episode_1/dataset_demo_0.hdf5

# 여러 에피소드를 한 HDF5 에 누적 (demo_0, demo_1, ...)
ros2 run ros2_mcap_recorder convert_mcap_to_hdf5 \
  --mcap logs/session_<ts>/episode_1/episode_1_0.mcap \
  --out  logs/session_<ts>/dataset.hdf5 --demo-name demo_0
ros2 run ros2_mcap_recorder convert_mcap_to_hdf5 \
  --mcap logs/session_<ts>/episode_2/episode_2_0.mcap \
  --out  logs/session_<ts>/dataset.hdf5 --demo-name demo_1 --append
```

### 1-3. 주요 CLI 옵션

| 인자 | 기본값 | 설명 |
| --- | --- | --- |
| `--session` | 최신 `session_*` 자동 선택 | 입력 세션 디렉터리 |
| `--mode` | `bundle` | `bundle` / `per-episode` |
| `--fps` | YAML 값 사용 | `sampling_rate` 런타임 override (`0`이면 resample 없이 원본 stamp 사용) |
| `--config` | 패키지의 `mcap_to_hdf5.yaml` | 매핑 YAML |
| `--out` | 모드별 기본 경로 | 출력 경로/디렉터리 |
| `--demo-name` | `demo_0` | `--mcap` / per-episode 모드 한정 |
| `--append` | `False` | `--mcap` 모드에서 기존 HDF5 에 누적 |
| `--logs-dir` | `./logs` | 세션 자동 탐색 부모 디렉터리 |

전체 옵션 목록과 `mcap_to_hdf5.yaml` 매핑(joint groups, poses, wrenches, command_streams, concat 으로 `actions`/`states` 생성) 작성법은 [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) README 의 §3-2, §3-3 을 참고하세요.

## 2. 데이터 확인

변환된 HDF5 의 그룹 구조·dataset shape·attribute 가 의도대로 채워졌는지 확인합니다. 별도의 프로그램 설치 없이 다음 온라인 뷰어로 빠르게 점검할 수 있습니다.

| 포맷 | 추천 뷰어 |
| --- | --- |
| MCAP | [Foxglove](https://app.foxglove.dev/) |
| HDF5 | [myHDF5](https://myhdf5.hdfgroup.org/) |
| LeRobot 데이터셋 | [LeRobot Visualizer](https://io-ai.tech/lerobot/) |

### MCAP 로컬 재생 (RViz2)

MCAP 파일은 `ros2 bag play` 로 토픽을 다시 publish 하면서 RViz2 로 시각적으로 확인할 수도 있습니다.

```bash
# 터미널 1 — MCAP 재생
ros2 bag play logs/session_<ts>/episode_1/episode_1_0.mcap

# 터미널 2 — RViz2 실행 후 카메라/TF/관절 토픽 추가
rviz2
```

`/robot_description` 과 `/tf_static` 은 latched 토픽으로 기록되어 있어 재생 시 자동으로 다시 publish 되므로 RViz2 에서 URDF 와 좌표계가 정상적으로 표시됩니다.

### myHDF5 확인 체크리스트

[myHDF5](https://myhdf5.hdfgroup.org/) 사용 시 다음을 확인하면 좋습니다.

- `data/demo_<i>/` 그룹이 에피소드 수만큼 생성되었는가
- 각 `obs/` 하위 dataset 의 첫 차원(시간축) 길이가 일관되는가
- `actions` / `states` 의 `layout`, `sources`, `source_dims` attribute 가 의도한 채널 순서대로 잡혔는가
- 이미지 dataset 의 `height` / `width` / `encoding` / `fps` 가 카메라 설정과 일치하는가
- 빈 dataset 이나 의도하지 않은 NaN 이 섞이지 않았는가

HDF5 그룹 레이아웃 전체 정의는 [데이터 포맷 → HDF5 스키마](../format/#hdf5-스키마) 참고.

## 3. LeRobot 포맷 변환 (선택)

LeRobot Visualizer / 워크플로우와의 호환이 필요하면 동일 패키지의 별도 변환기를 사용할 수 있습니다.

```bash
ros2 run ros2_mcap_recorder convert_mcap_to_lerobot \
  --mcap     logs/session_<ts>/episode_1/episode_1_0.mcap \
  --output-dir logs/session_<ts>/lerobot_dataset_ep0 \
  --episode-index 0
```

생성된 데이터셋은 [LeRobot Visualizer](https://io-ai.tech/lerobot/) 에 업로드해 직접 재생·검토할 수 있습니다.
