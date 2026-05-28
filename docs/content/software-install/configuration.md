+++
title = "설정 파일 / 환경 변수"
weight = 3
+++

## ROS 2 통신 설정 (UPC ↔ Teleop PC)

두 호스트는 같은 `192.168.2.0/24` 서브넷에서 ROS 2 DDS 가 아닌 **Zenoh RMW** 로 통신합니다. DDS 대비 NAT/방화벽 친화적이고, 멀티캐스트 없이 라우터 기반으로 디스커버리가 동작합니다.


## 환경 변수

| 변수 | 용도 | 예시 |
| --- | --- | --- |
| `ROS_DOMAIN_ID` | ROS2 도메인 분리 | `1` (기본) |
| `RMW_IMPLEMENTATION` | ROS2 RMW 구현체 | `rmw_zenoh_cpp` |


### 1. Zenoh RMW 설치 (양 호스트 모두)

```bash
sudo apt install -y ros-humble-rmw-zenoh-cpp
```

### 2. 환경 변수 (양 호스트 동일)

`~/.bashrc` 에 다음을 추가합니다.

```bash
export ROS_DOMAIN_ID=1
export RMW_IMPLEMENTATION=rmw_zenoh_cpp
```

### 3. Zenoh 라우터 등록 (systemd)

양 호스트 모두 Zenoh 라우터를 systemd 서비스로 등록해 부팅 시 자동 실행합니다.

- **RBY1 UPC**: 라우터를 `0.0.0.0:7447` 로 listen 만 수행
- **Teleop PC**: listen 과 동시에 UPC 라우터(`192.168.2.21:7447`) 로 connect

#### RBY1 UPC — `/etc/systemd/system/zenoh-router.service`

```ini
[Unit]
Description=ROS 2 Zenoh Router Daemon (Robot PC)
After=network.target network-online.target
Wants=network-online.target

[Service]
Type=simple
User=nvidia
Environment="ZENOH_CONFIG_OVERRIDE=listen/endpoints=[\"tcp/0.0.0.0:7447\"]"
ExecStart=/bin/bash -c "source /opt/ros/humble/setup.bash && export RMW_IMPLEMENTATION=rmw_zenoh_cpp && ros2 run rmw_zenoh_cpp rmw_zenohd"
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
```

#### Teleop PC — `/etc/systemd/system/zenoh-router.service`

```ini
[Unit]
Description=ROS 2 Zenoh Router Daemon (External PC)
After=network.target network-online.target
Wants=network-online.target

[Service]
Type=simple
User=cobot-ai
Environment="ZENOH_CONFIG_OVERRIDE=listen/endpoints=[\"tcp/0.0.0.0:7447\"];connect/endpoints=[\"tcp/192.168.2.21:7447\"]"
ExecStart=/bin/bash -c "source /opt/ros/humble/setup.bash && export RMW_IMPLEMENTATION=rmw_zenoh_cpp && ros2 run rmw_zenoh_cpp rmw_zenohd"
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
```

#### 서비스 활성화 (양 호스트 모두)

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now zenoh-router.service
sudo systemctl status zenoh-router.service
```

### 4. 통신 확인

ROS 2 표준 `demo_nodes_cpp` (rclcpp 기반 튜토리얼) 의 talker / listener 로 양 호스트 간 통신을 검증합니다.

RBY1 UPC 에서 talker 실행:

```bash
ros2 run demo_nodes_cpp talker
```

Teleop PC 에서 listener 실행:

```bash
ros2 run demo_nodes_cpp listener
```

Teleop PC 의 listener 콘솔에 `I heard: [Hello World: N]` 메시지가 출력되면 Zenoh 라우터를 통한 양방향 통신이 정상 동작하는 것입니다. 반대 방향(Teleop PC talker → UPC listener) 도 동일하게 확인할 수 있습니다.

