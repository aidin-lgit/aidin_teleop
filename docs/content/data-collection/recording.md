+++
title = "녹화 절차"
weight = 2
+++

녹화는 [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) 패키지의 `dynamic_mcap_recorder` 노드가 담당합니다. 노드를 띄워두고 두 개의 `std_srvs/SetBool` 서비스로 **세션** → **에피소드** 2-단계 구조로 녹화를 제어합니다.

## 1. 녹화 노드 기동

원격제어 통합 실행에서는 `teleop_bringup.launch.py` 가 로거를 함께 띄우므로 별도 실행이 필요 없습니다. 로거만 단독으로 띄울 때는 launch 를 쓰는 방식과 노드를 직접 실행하는 방식 두 가지가 있고, **기본 출력 경로가 서로 다릅니다.**

### 1-1. launch 로 실행 (권장)

```bash
ros2 launch aidin_rby1_teleop_bringup logger.launch.py
```

옵션을 지정해 실행할 수도 있습니다.

```bash
ros2 launch aidin_rby1_teleop_bringup logger.launch.py \
  config:=/path/to/topics.yaml \
  output_dir:=/mnt/data/teleop_logs \
  image_format:=raw
```

| 인자 | 기본값 | 설명 |
| --- | --- | --- |
| `config` | 패키지의 `config/topics.yaml` | 녹화 대상 토픽 목록 YAML. 기본 토픽 구성은 [데이터 포맷 → 기록 대상 토픽](../format/#기록-대상-토픽) 참고 |
| `output_dir` | **`~/data/mcap`** | 세션 디렉터리를 만들 부모 디렉터리 |
| `image_format` | `compressed` | 컬러 이미지 저장 형식 (`raw` / `compressed`) |

### 1-2. 노드를 직접 실행

```bash
ros2 run ros2_mcap_recorder mcap_recorder

# 옵션 지정
ros2 run ros2_mcap_recorder mcap_recorder \
  --config /path/to/topics.yaml \
  --output-dir ~/data/mcap \
  --image-format raw
```

이 방식의 `--output-dir` 기본값은 **`./logs`** (실행한 작업 디렉터리 기준) 입니다.

{{% notice style="warning" title="출력 경로 확인" %}}
launch 로 띄우면 `~/data/mcap/`, 노드를 직접 실행하면 `./logs/` 아래에 세션이 생성됩니다. 이후 [후처리 및 검증](../postprocessing/) 단계에서 변환 도구는 기본적으로 `./logs` 를 탐색하므로, launch 로 녹화한 세션은 `--logs-dir ~/data/mcap` 또는 `--session ~/data/mcap/session_<ts>` 를 명시해야 합니다.
{{% /notice %}}

토픽 자동 검출이 끝나면 노드 로그에 `모든 타겟 토픽의 타입을 성공적으로 찾았습니다!` 메시지가 출력됩니다.

## 2. 세션 / 에피소드 제어

원격제어 세션 중에는 `aidin_rby1_vive_teleop` 노드가 키보드 `a` / `c` / `b` (또는 발판 페달) 입력에 따라 아래 두 서비스를 자동으로 호출합니다. 전체 작업 흐름은 [원격제어 → 원격제어 및 녹화 작업 프로세스](../../teleoperation/operation/#원격제어-및-녹화-작업-프로세스) 절을 참고하세요.

| 서비스 | `data=true` | `data=false` |
| --- | --- | --- |
| `/dynamic_mcap_recorder/session_control` | 세션 시작 → `<output-dir>/session_YYYYMMDD_HHMMSS/` 생성, 에피소드 카운터 1로 리셋 | 세션 종료. 녹화 중이면 자동으로 에피소드를 먼저 닫음 |
| `/dynamic_mcap_recorder/record_control`  | 에피소드 녹화 시작 → 현재 세션 안에 `episode_<N>/` 생성 + MCAP Writer 오픈 | 에피소드 녹화 종료 + flush 저장. 다음 호출 시 `N+1` |

수동으로 호출해 디버깅하려면 다음과 같이 사용합니다.

```bash
# 세션 시작
ros2 service call /dynamic_mcap_recorder/session_control std_srvs/srv/SetBool "{data: true}"

# 에피소드 녹화 시작 / 종료
ros2 service call /dynamic_mcap_recorder/record_control  std_srvs/srv/SetBool "{data: true}"
ros2 service call /dynamic_mcap_recorder/record_control  std_srvs/srv/SetBool "{data: false}"

# 세션 종료
ros2 service call /dynamic_mcap_recorder/session_control std_srvs/srv/SetBool "{data: false}"
```

운영 규칙:

- `session_control(True)` 가 호출되지 않은 상태에서 `record_control(True)` 를 호출하면 `success=false` 로 거부됩니다.
- 같은 세션 안에서 `record_control` 을 toggle 할 때마다 `episode_1`, `episode_2`, … 순으로 폴더가 생성됩니다.
- 세션 종료 후 다시 `session_control(True)` 를 호출하면 새로운 `session_<timestamp>/` 가 만들어지고 에피소드 카운터는 1로 초기화됩니다.
- 노드를 `Ctrl+C` 로 종료할 때 녹화 중이었다면 자동으로 flush 후 닫힙니다.

## 3. 디렉터리 / 파일 구조

녹화가 끝난 뒤 출력 디렉터리는 다음과 같이 구성됩니다.

```text
~/data/mcap/                        # launch 실행 시. 노드 직접 실행 시에는 ./logs/
└── session_20260729_153000/
    ├── episode_1/
    │   ├── episode_1_0.mcap
    │   └── metadata.yaml
    ├── episode_2/
    │   ├── episode_2_0.mcap
    │   └── metadata.yaml
    └── ...
```

- `session_<YYYYMMDD>_<HHMMSS>/` 는 세션 시작 시각으로 자동 명명되며, 사용자 지정 불가.
- 각 `episode_<N>/` 폴더 안에는 MCAP 파일과 rosbag2 가 생성하는 `metadata.yaml` 이 함께 저장됩니다.
- HDF5 일괄 변환이 끝난 뒤에는 동일 세션 폴더 안에 `dataset.hdf5` 가 추가됩니다 (`bundle` 모드).

## 4. 토픽 화이트리스트 변경

녹화 대상 토픽을 추가·제거하려면 `ros2_mcap_recorder/config/topics.yaml` 을 수정한 뒤 패키지를 재빌드합니다.

```yaml
topics:
  - /joint_states
  - /tf_static
  - /robot_description
  - /camera/head/color/image_raw
  # ...

# 발행자가 TRANSIENT_LOCAL 이 아니어도 시작 stamp 에 한 번 더 기록되도록 강제
latched_topics:
  - /some/custom_latched_topic
```

- `latched_topics` 에 명시된 토픽은 녹화 시작 시점 stamp 로 한 번 더 기록되므로 변환기에서 항상 첫 행에 잡힙니다.
- `topics.yaml` 에 없는 토픽은 MCAP 에 기록되지 않습니다 — 변환에서 필요한 토픽은 모두 포함시키세요.

자세한 QoS 자동 검출·latched 처리·HDF5/LeRobot 변환 매핑은 [`ros2_mcap_recorder`](https://github.com/aidin-lgit/ros2_mcap_recorder) README 를 참고하세요.
