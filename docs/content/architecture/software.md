+++
title = "소프트웨어 아키텍처"
weight = 2
+++

![System Architecture](/images/system_architecture.png)

본 시스템의 ROS 2 스택은 **Description → Hardware Interface → Controller** 세 계층으로 구성되며, 위로 텔레오퍼레이션 패키지 묶음이 얹히는 구조입니다. 모든 패키지는 ROS 2 Humble + ros2_control 위에서 동작합니다.

## ROS2 패키지 구성

### 1. Description (URDF / xacro)


| 패키지                      | 역할                                                                          |
| ------------------------ | --------------------------------------------------------------------------- |
| `aidin_hand_description` | 5지 AIDIN Hand 의 URDF / xacro / 메쉬 / RViz · 좌·우 손 매크로, ros2_control 매크로      |
| `aidin_rby1_description` | RBY1 + AIDIN Hand 통합 모델 (URDF / xacro / Mujoco MJCF / Isaac Sim USD / RViz) |


### 2. Hardware Interface (ros2_control SystemInterface 플러그인)

URDF 의 `<ros2_control>` 블록에서 로드되며 자체 노드는 띄우지 않습니다.


| 패키지                   | 제공 플러그인                                                                                                                                                                                                          | 비고                                                    |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `aidin_hand_hardware` | `EtherlabDriver` (실기, IgH EtherCAT + CiA-402 + Homing 시퀀스 내장, 1 kHz RT 루프) / `EthercatDriver` (`ethercat_interface` 기반 일반형) / `AidinHandVirtualHardware` (가상)                                                    | 5 손가락 × ELMO 슬레이브 1개 × 3축. axis1 에 F 센서, axis2 에 T 센서 |
| `aidin_rby1_hardware` | `RBY1HardwareInterface` (실기, [`rby1-sdk`](https://github.com/RainbowRobotics/rby1-sdk) gRPC, 24 DoF + 손목 FT + EE/torso/head pose, 위치/임피던스 + Homing/E-stop) / `AidinRBY1VirtualHardware` (가상) | 별도 `rby1_description` (Rainbow 측) URDF 런타임 로드 |


### 3. Controller (ros2_control 컨트롤러)


| 패키지                      | 주요 컨트롤러                                                                                                                                                                                                                                                                                                                                   | 설명                                                                                                                                                                                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `aidin_hand_controllers` | `ActuatorController` → `HandKinematicsController` → `JointCommandController` ( `ChainableControllerInterface`)                                                                                                                                                                                                                          | 사용자 관절 명령 → 손가락 운동학 → 액추에이터 인코더로 chain 전파. 손가락 F-T 브로드캐스터 동반                                                                              |
| `aidin_rby1_controller`  | **Joint layer**: `JointCommandController` (필수, terminal) **Cartesian layer** (택1): `BimanualIKController` / `WholebodyIKController` / `CartesianAdmittanceController` / `WholebodyCartesianAdmittanceController` **Mobile layer** (옵션): `diff_drive_controller` **Broadcaster**: joint_state, EE pose, torso/head pose, wrist FT, hand FT |  launch 인자로 Cartesian 컨트롤러 자동 선택 |


### 4. Teleoperation (상위 스택)


| 패키지                 | 역할                                                                                                                                                                                                                                                                                                |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `aidin_rby1_teleop` | VR HMD · VIVE Tracker · MANUS Glove 입력을 단일 ref frame 으로 정합 후 위 Cartesian / Joint / Hand 컨트롤러로 송출, 동시에 MCAP 으로 녹화 |


## 런타임 데이터 플로우 (텔레옵 → 로봇)

```mermaid
flowchart LR
  subgraph INPUT["입력 디바이스 (작업자측)"]
    QUEST["Meta Quest<br/>(head pose · ego video)"]
    VIVE["VIVE Tracker × 3<br/>(좌·우 손목 · 허리)"]
    MANUS["MANUS Glove<br/>(손가락 관절)"]
  end

  subgraph TELE["aidin_rby1_teleop"]
    VR["aidin_rby1_vr_teleop"]
    VT["aidin_rby1_vive_teleop"]
    MG["aidin_manus"]
    REC["ros2_mcap_recorder"]
  end

  subgraph CTRL["aidin_rby1_controller (Cartesian + Joint Layer)"]
    CART["BimanualIK /<br/>WholebodyIK"]
    JNT["JointCommandController"]
  end

  HCTRL["aidin_hand_controllers<br/>(Actuator → Kinematics → JointCmd)"]

  subgraph HW["ros2_control_node + Hardware"]
    RHW["aidin_rby1_hardware<br/>→ RBY1 (rby1-sdk)"]
    HHW["aidin_hand_hardware<br/>→ EtherCAT"]
  end

  QUEST --> VR
  VIVE --> VT
  MANUS --> MG

  VR -- "PoseArray (head)" --> CART
  VT -- "PoseArray (좌·우 EE + torso)" --> CART
  MG -- "JointState (손가락)" --> HCTRL

  CART -- "chained ref<br/>(joint target)" --> JNT
  JNT  -- "position cmd" --> RHW
  HCTRL -- "axis position cmd" --> HHW

  RHW -. "joint state · wrist FT · EE pose" .-> CART
  HHW -. "joint state · finger FT" .-> HCTRL

  REC -. "topic 일괄 기록" .- TELE
  REC -. "topic 일괄 기록" .- CTRL
  REC -. "topic 일괄 기록" .- HW
```



## 주요 인터페이스


| 입력 토픽 (외부 → 컨트롤러)                                    | 타입                                              | 흐름                                      |
| ---------------------------------------------------- | ----------------------------------------------- | --------------------------------------- |
| `/aidin_rby1_bimanual_controller/pose_array_command` | `geometry_msgs/PoseArray` (좌 EE / 우 EE / torso) | VIVE Tracker → bimanual IK              |
| `/aidin_rby1_joint_controller/joint_state_command`   | `sensor_msgs/JointState`                        | joint-space 직접 명령|
| `/aidin_rby1_mobile_controller/cmd_vel_unstamped`    | `geometry_msgs/Twist`                           | 모바일 베이스 차동 구동                           |



| 출력 토픽 (state · feedback)                        | 타입                                          |
| ----------------------------------------------- | ------------------------------------------- |
| `/joint_states`                                 | `sensor_msgs/JointState`                    |
| `/{side}_eef_pose_broadcaster/pose`             | `geometry_msgs/PoseStamped`                 |
| `/{side}_wrist_ft_sensor_broadcaster/wrench`    | `geometry_msgs/WrenchStamped`               |
| `/{side}_ft_sensor_broadcaster/wrench`          | `sensor_msgs/MultiDOFJointState` (5 손가락 묶음) |
| `/{side}_{finger}_ft_sensor_broadcaster/wrench` | `geometry_msgs/WrenchStamped` (손가락별)        |



| 서비스                                       | 타입                 | 용도                                                                        |
| ----------------------------------------- | ------------------ | ------------------------------------------------------------------------- |
| `/{cartesian_controller}/home_to_initial` | `std_srvs/Trigger` | 로봇 자세 초기화 서비스 |
| `/rby1_hw_node/emergency_stop`            | `std_srvs/Trigger` | 로봇 긴급 정지 서비스 |

