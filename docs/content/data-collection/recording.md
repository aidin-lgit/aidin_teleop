+++
title = "녹화 절차"
weight = 2
+++

## 한 에피소드의 흐름

1. 텔레옵 구동 (참고: [원격제어 → 부팅 시퀀스](../../teleoperation/launch-sequence/))
2. 녹화 노드 시작
   ```bash
   ros2 launch aidin_rby1_teleop record.launch.py \
       episode_name:=pick_red_block_0001 \
       operator:=jm
   ```
3. 텔레옵 활성화 (`Engage`)
4. 작업 수행 (5~30초 권장)
5. 텔레옵 비활성화 (`Disengage`)
6. 녹화 노드 정지 → MCAP 파일 닫힘
7. 결과 검토 (다음 페이지 참고)

## 파일 명명 규칙

`<task>_<variant>_<index>_<YYYYMMDD>_<operator>.mcap`

예: `pick_red_block_0001_20260601_jm.mcap`

## 라벨링 / 메타데이터

녹화 시 launch 인자로 들어간 정보가 MCAP 의 attachment 또는 사이드카 yaml 로 저장됩니다.

```yaml
# /data/raw/pick_red_block_0001_20260601_jm.yaml
task: pick_red_block
variant: default
operator: jm
notes: "처음 시도, 블록 위치 좌측 끝"
success: true
```

> 후처리 단계에서 yaml 이 함께 HDF5 의 `/meta/` 그룹으로 병합됩니다.
