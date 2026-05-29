+++
title = "원격제어 실행 및 조작법"
weight = 3
+++

## 원격제어 실행

Teleop PC 에서 다음 launch 를 실행하면 트래커 입력을 받아 로봇으로 송출하는 노드가 기동됩니다.

```bash
ros2 launch aidin_rby1_teleop_bringup teleop_control.launch.py
```

머리 자세 제어와 Meta Quest HMD 영상 수신을 함께 사용하는 경우 `vr_control:=true` 인자를 추가합니다.

```bash
ros2 launch aidin_rby1_teleop_bringup teleop_control.launch.py vr_control:=true
```

시작/일시정지/긴급 정지는 키보드 `a` / `c` / `b` 또는 발판 페달로 제어합니다. 상태 전이 상세는 [Quick Start §5](../../introduction/quick-start/#5-원격-제어-및-데이터-로깅-시작) 또는 [`aidin_rby1_vive_teleop`](https://github.com/aidin-lgit/aidin_rby1_vive_teleop) README 참고.

## 원격제어 및 녹화 작업 프로세스

원격제어 시작부터 종료까지의 전체 흐름은 다음과 같이 키보드(또는 발판 페달)로 진행됩니다. 한글 IME 상태에서도 동일하게 인식됩니다 (`ㅁ` = `a`, `ㅊ` = `c`, `ㅠ` = `b`).

1. **1차 `a` — 세션 시작 및 로봇 초기화**
   녹화 세션이 열리고 (`session_control(True)`), 로봇이 초기화 자세로 Homing 합니다. 이 단계까지는 아직 트래커 입력이 로봇으로 송출되지 않습니다.

2. **2차 `a` — 에피소드 녹화 시작 및 로봇 제어 시작**
   세션 안에 새로운 `episode_<N>/` 폴더와 MCAP 파일이 생성되고 (`record_control(True)`), 작업자의 트래커·HMD 입력이 로봇 명령으로 송출되기 시작합니다 (publish ON). 이 시점에 허리(및 머리) reference frame 이 캡처됩니다.

3. **`c` — 에피소드 종료**
   현재 에피소드의 녹화가 닫히고 로봇 제어가 정지되며, 초기화 자세로 복귀합니다. 세션 자체는 유지되므로 동일 세션 내에서 다음 에피소드를 이어 녹화할 수 있습니다.

4. **추가 에피소드** — 2단계(`a` 한 번 더)부터 반복하면 `episode_2`, `episode_3`, … 가 같은 세션 폴더 안에 생성됩니다.

5. **세션 종료** — 모든 에피소드 작업이 끝나면 한 번 더 `c` 를 눌러 세션을 닫고 노드를 종료합니다.

6. **`b` — 긴급 정지** — 위 어느 단계에서든 `b` (또는 가운데 페달) 를 누르면 `emergency_stop` 호출 + 세션·녹화 OFF + 노드 종료까지 한 번에 수행됩니다. 자세한 동작은 [안전 절차 / E-Stop](../safety/#원격-제어시-e-stop) 참고.

이렇게 수집된 세션 폴더는 [Quick Start §6](../../introduction/quick-start/#6-세션--hdf5-변환) 의 `convert_session_to_hdf5` 명령으로 RoboMimic 호환 HDF5 로 일괄 변환할 수 있습니다.

## 제어 시작 전 작업자의 시작 자세

{{% notice style="warning" title="작업 시작 자세 — 활성화 직전 반드시 확인" %}}
시작 시점의 허리(및 머리) 트래커 자세가 reference frame 으로 캡처되므로, 자세가 어긋난 상태로 활성화하면 의도하지 않은 로봇 움직임이 발생할 수 있습니다.

- 사용자는 **로봇의 초기 자세에 맞춰** 자신의 자세를 잡은 뒤 원격제어를 시작합니다. 사용자가 로봇과 같은 방향을 바라볼 필요는 없습니다 (어느 방향을 향해 서 있어도 로봇 `base_footprint` 기준의 동일한 움직임이 재현됨).
- 처음에는 **양손을 가슴 앞으로 모은 중립 자세**에서 시작하는 것을 권장합니다.
- **머리 자세 제어(`vr_control:=true`) 모드** 로 동작할 경우, 시작 시점에는 **사용자가 정면을 바라본 상태**에서 활성화해야 머리 제어 reference 가 올바르게 잡힙니다.
{{% /notice %}}

## 머리 카메라 제어 시 이미지 수신 방법

`vr_control:=true` 모드로 실행한 경우, 머리 자세 송신과 로봇 카메라 영상 수신은 HMD 브라우저를 통해 이루어집니다. 다음 절차에 따라 스트리밍 세션을 시작하세요.

1. HMD 브라우저 앱에서 Teleop PC 의 주소로 접속합니다 (defaut: https://192.168.2.31:8012).

   ![HMD 브라우저 접속 시도](/images/hmd_connection.png?width=400px)

2. 페이지가 로드되면 데이터(센서/카메라) 접근 권한 요청이 표시됩니다. **허용**을 눌러 권한을 수락합니다.

   ![HMD 권한 수락](/images/hmd_allow.png?width=400px)

3. 권한 수락 후 스트리밍 서버(default: https://vuer.ai/?ws=wss://192.168.2.31:8012)로 접속하여 영상 확인 후 우측 상단의 **VR 시작하기** 버튼을 클릭합니다. 이 시점부터 HMD 의 머리 자세가 Teleop PC 로 송신되고, 로봇 카메라 영상이 HMD 로 수신됩니다.

   ![HMD VR 시작하기 버튼](/images/hmd_stream_server.png?width=400px)



| 파라미터 | 기본값 | 권장 조정 범위 | 설명 |
| --- | --- | --- | --- |
| `vuer_image_scale` | `0.5` | `0.3 ~ 0.4` | HMD 로 보낼 해상도 스케일 (`0.2 ~ 1.0`) |
| `vuer_jpeg_quality` | `70` | `55 ~ 65` | image 모드 JPEG 품질 (`30 ~ 95`) |
| `vuer_fps` | `30.0` | `15 ~ 20` | image 모드 송신 주기 Hz (`5 ~ 60`) |

조정 예시:

```bash
ros2 run aidin_rby1_vr_teleop <node_name> \
  -p vuer_image_scale:=0.35 \
  -p vuer_jpeg_quality:=55 \
  -p vuer_fps:=20
```


## 운영 팁

- 원격제어 조작자의 움직임이 VIVE Base station 범위 안에 위치
- 로봇 움직임이 사용자 움직임과 다르게 움직이면 실행중인 프로그램 재시작
