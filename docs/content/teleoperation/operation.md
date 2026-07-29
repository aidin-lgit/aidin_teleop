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

허리까지 트래커로 제어하려면 `control_torso:=true` 를 추가합니다 (기본값은 시작 시점 자세로 고정).

```bash
ros2 launch aidin_rby1_teleop_bringup teleop_control.launch.py \
    vr_control:=true control_torso:=true
```

주요 launch 인자는 다음과 같습니다. 트래커·명령 토픽 이름(`left_hand_topic`, `pose_command_topic` 등)도 같은 방식으로 `aidin_rby1_vive_teleop` 에 위임됩니다.

| 인자 | 기본값 | 설명 |
| --- | --- | --- |
| `vr_control` | `false` | 머리 제어(`vr_controller.py`) 실행 여부 |
| `control_torso` | `false` | 허리를 허리 트래커로 제어 |
| `keyboard_mode` | `global` | 키 입력 소스 — `global`(전역 후킹) / `terminal` / `off` |
| `auto_start` | `false` | `true` 면 기동과 동시에 세션 열기 + 녹화 + 발행까지 진행 |
| `rate_hz` | `60.0` | 팔·허리 명령 발행 주기 |
| `vr_rate_hz` | `60.0` | 머리 제어 주기 |
| `hmd_pose_topic` | `/vr/hmd_pose` | 머리 제어 입력 토픽 (CloudXR 발행) |
| `pose_command_topic` | `/aidin_rby1_bimanual_controller/pose_array_command` | 양팔·허리 PoseArray 명령 출력 |

시작/에피소드 종료/긴급 정지는 키보드 `a` / `c` / `b` 또는 발판 페달로 제어합니다. 상태 전이 상세는 아래 [원격제어 및 녹화 작업 프로세스](#원격제어-및-녹화-작업-프로세스) 또는 [`aidin_rby1_vive_teleop`](https://github.com/aidin-lgit/aidin_rby1_vive_teleop) README 참고.


## 원격제어 및 녹화 작업 프로세스

원격제어 시작부터 종료까지의 전체 흐름은 다음과 같이 키보드(또는 발판 페달)로 진행됩니다. 한글 IME 상태에서도 동일하게 인식됩니다 (`ㅁ` = `a`, `ㅊ` = `c`, `ㅠ` = `b`).

노드는 `IDLE → SESSION → TELEOP → HOMING` 네 상태를 가지며, `a` 로 앞으로 진행하고 `c` 로 세션을 닫습니다.

1. **1차 `a` — 세션 시작 및 로봇 초기화** (IDLE → SESSION)
   녹화 세션이 열리고 (`session_control(True)`), 로봇이 초기화 자세로 Homing 합니다. 이 단계까지는 아직 트래커 입력이 로봇으로 송출되지 않습니다.

2. **2차 `a` — 에피소드 녹화 시작 및 로봇 제어 시작** (SESSION → TELEOP)
   세션 안에 새로운 `episode_<N>/` 폴더와 MCAP 파일이 생성되고 (`record_control(True)`), 작업자의 트래커·HMD 입력이 로봇 명령으로 송출되기 시작합니다 (publish ON). 이 시점에 허리(및 머리) reference frame 이 캡처됩니다.

3. **`a` — 에피소드 종료 (세션 유지)** (TELEOP → HOMING → SESSION)
   현재 에피소드의 녹화가 닫히고 로봇 제어가 정지되며, 초기화 자세로 복귀합니다. **세션은 유지**되므로 동일 세션 폴더 안에서 다음 에피소드를 이어 녹화할 수 있습니다.

4. **추가 에피소드** — 2단계(`a`)부터 반복하면 `episode_2`, `episode_3`, … 가 같은 세션 폴더 안에 생성됩니다.

5. **`c` — 세션 종료** (TELEOP 또는 SESSION → IDLE)
   모든 에피소드 작업이 끝나면 `c` 로 세션을 닫습니다. TELEOP 중에 눌렀다면 녹화 종료 + 세션 닫기 + 호밍이 한 번에 수행됩니다. IDLE 에서 `c` 를 한 번 더 누르면 노드가 종료됩니다.

6. **`b` — 긴급 정지** — 위 어느 단계에서든 `b` (또는 가운데 페달) 를 누르면 `emergency_stop` 호출 + 세션·녹화 OFF + 노드 종료까지 한 번에 수행됩니다. 자세한 동작은 [안전 절차 / E-Stop](../safety/#원격-제어시-e-stop) 참고.

{{% notice style="warning" title="에피소드 반복은 `a`, 세션 종료는 `c`" %}}
에피소드를 이어서 수집할 때 3단계에서 `c` 를 누르면 **세션까지 닫힙니다**. 이 상태에서 다시 `a` 를 누르면 새로운 `session_<timestamp>/` 폴더가 생성되어 에피소드가 세션마다 흩어지고, 세션 일괄 변환(`convert_session_to_hdf5`) 시 한 번에 묶이지 않습니다.

**같은 세션에 에피소드를 누적하려면 3단계에서 반드시 `a`** 를 사용하세요.
{{% /notice %}}

| 키 | IDLE | SESSION | TELEOP | HOMING |
| --- | --- | --- | --- | --- |
| `a` / `ㅁ` | 세션 열기 → SESSION | 녹화 + 명령 발행 시작 → TELEOP | 안전 정지 상태면 `resume`, 아니면 녹화 종료 + 호밍 (**세션 유지**) → SESSION | 무시 |
| `c` / `ㅊ` | 프로그램 종료 | 세션 닫기 → IDLE | 녹화 종료 + 세션 닫기 + 호밍 → IDLE | 무시 |
| `b` / `ㅠ` | 비상 정지 + 종료 | 비상 정지 + 종료 | 비상 정지 + 녹화 종료 + 세션 닫기 + 종료 | 비상 정지 + 종료 |

호밍 중에는 `a` / `c` 입력이 무시됩니다.

## 제어기 입력 안전 검사 (안전 정지)

컨트롤러에는 **입력 안전 검사(input safety)** 가 내장되어 있습니다. 트래커가 순간적으로 튀거나 사용자가 너무 빠르게 움직여 비정상적인 명령이 들어오면, 로봇이 그 명령을 그대로 따라 급격히 움직이는 것을 막기 위해 **전체를 정지 상태로 latch** 합니다.

정지된 뒤에는 명령을 계속 보내도 로봇이 움직이지 않으며, **반드시 해제(resume) 를 해야** 제어가 재개됩니다.

### 정지 조건

두 가지 조건이 정지를 유발합니다. 정지 시 원인·해당 limb·측정값이 컨트롤러 로그에 남습니다.

| 구분 | 조건 | 의미 |
| --- | --- | --- |
| **JUMP** | 한 제어 주기 안의 **위치 변화**가 `max_delta_pos` 초과 | 손 목표가 한 프레임에 순간 이동 (트래커 튐, occlusion 후 복귀 등) |
| **JUMP** | 한 제어 주기 안의 **회전 변화**가 `max_delta_ori_deg` 초과 | 손목 자세가 한 프레임에 급회전 |
| **FAST** | EEF **이동 속도**가 `max_input_vel` 을 `vel_over_frames` 회 연속 초과 | 사용자가 팔을 지나치게 빠르게 움직임 |

{{% notice style="info" title="입력이 끊기는 것은 정지가 아닙니다" %}}
`input_timeout_ms`(기본 200 ms) 동안 명령이 들어오지 않는 경우는 **정지가 아닙니다.** 경고만 남기고 마지막 명령을 유지하며, 입력이 돌아오면 그대로 이어집니다. latch 되는 것은 **JUMP / FAST 뿐**입니다.
{{% /notice %}}

### 정지 여부 확인

컨트롤러의 latched 상태 토픽으로 확인합니다. 늦게 구독해도 마지막 상태를 즉시 받습니다.

```bash
# 현재 상태 1회 조회
ros2 topic echo --once --qos-durability transient_local --qos-reliability reliable \
    /aidin_rby1_bimanual_controller/safety_status

# 상태 변화 실시간 감시
ros2 topic echo /aidin_rby1_bimanual_controller/safety_status
```

`diagnostic_msgs/DiagnosticStatus` 의 `level` 로 판별합니다.

| `level` | 의미 | 조치 |
| --- | --- | --- |
| 0 (OK) | 정상 추종 중 | — |
| 1 (WARN) | 입력 끊김, 마지막 명령 유지 | 자동 복귀. 트래커 연결 확인 |
| 2 (ERROR) | **안전 정지 latch** | 해제 필요 (아래) |

`values[]` 에 `stopped` / `reason` / `limb` / `delta_pos_m` / `delta_ori_deg` / `velocity_mps` / `input_age_ms` 가 함께 담기므로, 어떤 값이 상한을 넘겨 멈췄는지 바로 확인할 수 있습니다 (트립 수치는 정지 중에만 유효).

상태는 변화 시 즉시, 그 외에는 `status_rate_hz`(기본 10 Hz) 주기로 발행됩니다. **토픽이 조용해지면 컨트롤러 자체가 내려간 것**입니다.

### 해제 방법

원격제어 중이라면 **키보드 `a`** 가 가장 간단합니다. TELEOP 상태에서 컨트롤러가 정지 상태이면, `a` 는 에피소드를 끝내지 않고 해당 컨트롤러의 `resume` 를 호출해 **같은 에피소드에서 제어를 이어갑니다** (녹화·세션 그대로 유지). 정지 상태가 아니라면 평소대로 에피소드 종료 + 호밍으로 동작합니다.

수동으로 해제할 수도 있습니다.

```bash
# 정지 해제 (정지 상태가 아니면 success: false 로 거부됨)
ros2 service call /aidin_rby1_bimanual_controller/resume std_srvs/srv/Trigger

# 수동 호밍
ros2 service call /aidin_rby1_bimanual_controller/home_to_initial std_srvs/srv/Trigger
```

해제 시 명령 기준(baseline)이 **현재 자세로 다시 잡힙니다.** 정지 이전에 누적된 오프셋은 이어지지 않으므로 로봇이 튀지 않고 이어집니다.

{{% notice style="warning" title="상태 확인용으로 `resume` 를 호출하지 마세요" %}}
`~/resume` 는 조회 명령이 아닙니다. 정지 중이라면 실제로 정지를 해제하고 명령 기준을 재설정하므로, 상태 확인은 반드시 `~/safety_status` 토픽으로 하세요.
{{% /notice %}}

### 임계값 조정 (config 수정)

임계값은 launch 인자가 아니라 **컨트롤러 yaml 파일**에서 수정합니다. 로봇 PC(RBY1 UPC)의 다음 경로에 있습니다.

```text
~/Workspace/aidin_ws/src/aidin_rby1_controller/config/<활성_컨트롤러>.yaml
```

`ik_config.input_safety` 블록입니다.

```yaml
/**:
  aidin_rby1_bimanual_controller:
    ros__parameters:
      ik_config:
        input_safety:
          enabled: true             # false 면 검사 전체를 끈다
          max_delta_pos: 0.1        # [m]   JUMP: 프레임당 위치 변화 상한
          max_delta_ori_deg: 20.0   # [deg] JUMP: 프레임당 회전 변화 상한
          max_input_vel: 1.5        # [m/s] FAST: EEF 속도 상한 (0 = 사용 안 함)
          vel_over_frames: 2        #       FAST: 연속 초과 프레임 수
          input_timeout_ms: 200     # 입력 끊김 판정 (정지 아님)
          status_rate_hz: 10.0      # 상태 발행 주기 [Hz]. 0 = 변화 시에만
          debug: false              # true 면 프레임별 변화량/속도 로그 출력
```

**임계값은 컨트롤러마다 다릅니다.** 활성 컨트롤러에 해당하는 파일을 수정해야 합니다 ([어떤 컨트롤러가 활성인지](../launch-sequence/#cartesian-컨트롤러-선택)).

| 컨트롤러 yaml | `max_delta_pos` | `max_delta_ori_deg` | `max_input_vel` |
| --- | --- | --- | --- |
| `aidin_rby1_bimanual_controller.yaml` | 0.1 m | 20.0° | 1.5 m/s |
| `aidin_rby1_wholebody_controller.yaml` | 0.05 m | 15.0° | 1.0 m/s |
| `aidin_rby1_cartesian_admittance_controller.yaml` | 0.05 m | 15.0° | 1.0 m/s |
| `aidin_rby1_wholebody_cartesian_admittance_controller.yaml` | 0.01 m | 15.0° | 0.5 m/s |

수정 후에는 컨트롤러를 **재시작**해야 반영됩니다 (`robot_launch.sh` 를 `Ctrl+C` 로 내리고 다시 실행). 워크스페이스가 `--symlink-install` 로 빌드되어 있으면 재빌드는 필요 없습니다.

{{% notice style="warning" title="임계값을 올릴 때" %}}
정지가 너무 자주 걸린다면 먼저 `debug: true` 로 두고 **실제 변화량·속도 값을 로그로 확인한 뒤** 그 값을 근거로 상한을 올리세요. 임계값을 무작정 크게 잡거나 `enabled: false` 로 검사를 끄면, 트래커가 튀었을 때 로봇이 그 명령을 그대로 추종해 급격하게 움직입니다.

정지가 잦은 근본 원인이 트래커 품질일 수도 있습니다 — lighthouse 시야 가림, 반사면, 트래커 배터리를 함께 점검하세요.
{{% /notice %}}

이렇게 수집된 세션 폴더는 [데이터 수집 → 후처리 및 검증](../../data-collection/postprocessing/) 의 `convert_session_to_hdf5` 명령으로 RoboMimic 호환 HDF5 로 일괄 변환할 수 있습니다.

## 제어 시작 전 작업자의 시작 자세

{{% notice style="warning" title="작업 시작 자세 — 활성화 직전 반드시 확인" %}}
시작 시점의 허리(및 머리) 트래커 자세가 reference frame 으로 캡처되므로, 자세가 어긋난 상태로 활성화하면 의도하지 않은 로봇 움직임이 발생할 수 있습니다.

- 사용자는 **로봇의 초기 자세에 맞춰** 자신의 자세를 잡은 뒤 원격제어를 시작합니다. 사용자가 로봇과 같은 방향을 바라볼 필요는 없습니다 (어느 방향을 향해 서 있어도 로봇 `base_footprint` 기준의 동일한 움직임이 재현됨).
- 처음에는 **양손을 가슴 앞으로 모은 중립 자세**에서 시작하는 것을 권장합니다.
- **머리 자세 제어(`vr_control:=true`) 모드** 로 동작할 경우, 시작 시점에는 **사용자가 정면을 바라본 상태**에서 활성화해야 머리 제어 reference 가 올바르게 잡힙니다.
{{% /notice %}}

## 머리 카메라 제어 시 이미지 수신 방법

`vr_control:=true` 모드로 실행한 경우, 머리 자세 송신과 로봇 카메라 영상 수신은 **NVIDIA CloudXR** 스택과 HMD 브라우저(WebXR)를 통해 이루어집니다. 별도 앱 설치 없이 Quest 브라우저에서 URL 접속만으로 동작합니다.

인터페이스 bringup 이 `use_vr_stream:=true`(기본값) 로 실행되어 CloudXR 스택이 올라온 상태에서, 다음 절차로 스트리밍 세션을 시작하세요.

1. HMD 브라우저에서 **WSS 프록시 주소**에 먼저 접속해 인증서 경고를 수락합니다. 이 단계는 시그널링 채널의 TLS 를 헤드셋에 신뢰 등록하는 과정으로, 건너뛰면 다음 단계에서 연결이 실패합니다.

   ```
   https://<Teleop PC IP>:48322
   ```

2. 이어서 **web client 페이지**에 접속해 페이지 인증서 경고를 수락합니다.

   ```
   https://<Teleop PC IP>:8080
   ```

3. 페이지에서 다음을 설정합니다.
   - **Server IP** — Teleop PC IP (`endpoint_ip` 로 지정한 값)
   - **Port** — 비워 둠 (기본 `48322` 사용)
   - **Immersive Mode** — `VR`

4. **CONNECT** → **Enter XR** 를 눌러 immersive 로 진입합니다. 이 시점부터 HMD 의 머리 자세가 Teleop PC 로 송신되고(`/vr/hmd_pose`), 로봇 카메라 영상이 HMD 로 수신됩니다.

{{% notice style="warning" title="HTTPS 인증서 경고" %}}
CloudXR 스택은 로컬 자체 서명 인증서를 사용하므로 두 주소 모두에서 브라우저 보안 경고가 표시됩니다. **Advanced** (고급) → **Proceed to `<주소>` (unsafe)** 로 진행하세요. 인증서의 SAN 에 서버 IP 가 포함되어 있어야 헤드셋이 신뢰하므로, 발급 시 `mkcert -cert-file cert.pem -key-file key.pem <서버 IP> localhost` 형태로 IP 를 포함해야 합니다 ([`aidin_rby1_vr_teleop`](https://github.com/aidin-lgit/aidin_rby1_vr_teleop) README §2.6).

`48322` → `8080` **순서**로 두 번 수락해야 합니다.
{{% /notice %}}

### CloudXR 런치 인자

`teleop_bringup.launch.py` 는 `use_vr_stream` / `endpoint_ip` 두 인자를 위임하며, 나머지는 `cloudxr_teleop.launch.py` 의 기본값을 그대로 사용합니다. 단독 실행 시 조정할 수 있는 인자는 다음과 같습니다.

| 인자 | 기본값 | 설명 |
| --- | --- | --- |
| `endpoint_ip` | `""` (자동) | 헤드셋이 접속할 서버 IP. 인증서 SAN 과 일치시킬 것 |
| `enable_video` | `true` | `false` = 영상 없이 머리·손 입력만 사용 |
| `camera_location` | `local` | `local` = 이 PC 의 결합 stereo 토픽 / `remote` = 다른 PC 의 좌·우 개별 토픽 구독·결합 |
| `image_transport` | `compressed` | ZED 구독 transport. `raw` 는 무손실(화질↑, 대역폭↑) |
| `start_webclient` | `true` | web client dev-server 동시 실행 여부 |
| `signaling_port` | `49100` | CloudXR 시그널링 WS 포트 |
| `wss_port` | `48322` | WSS 프록시 포트 (위 1단계 주소) |
| `cert` / `key` | `script/image_stream/{cert,key}.pem` | WSS TLS 인증서 / 키 |

조정 예시:

```bash
# 영상 없이 입력만 (CloudXR 인코딩 부하 제거, 연결 점검용)
ros2 launch aidin_rby1_vr_teleop cloudxr_teleop.launch.py \
  endpoint_ip:=192.168.2.31 enable_video:=false

# ZED 가 다른 PC(카메라 보드)에 붙어 있고 무손실 영상을 쓰는 경우
ros2 launch aidin_rby1_vr_teleop cloudxr_teleop.launch.py \
  endpoint_ip:=192.168.2.31 camera_location:=remote image_transport:=raw
```

전송 해상도·비트레이트 조정은 [`webclient/README.md`](https://github.com/aidin-lgit/aidin_rby1_vr_teleop/blob/main/webclient/README.md) 를 참고하세요. 헤드셋과 서버가 다른 서브넷 / NAT 너머에 있어 영상이 나오지 않으면 TURN 서버(coturn) 설정이 필요합니다.


## 운영 팁

- 원격제어 조작자의 움직임이 VIVE Base station 범위 안에 위치
- 로봇 움직임이 사용자 움직임과 다르게 움직이면 실행중인 프로그램 재시작
