+++
title = "후처리 및 검증"
weight = 3
+++

## 변환 후 검증 체크리스트

- [ ] HDF5 의 `/obs/joint_pos` 길이가 기록된 duration 과 일치하는가
- [ ] 이미지 토픽과 상태 토픽의 시간 동기 오차가 허용 범위 (예: 10ms) 이내인가
- [ ] 결측 (NaN, dropout) 비율이 임계 이하인가
- [ ] 텔레옵 비활성화 구간이 정확히 제거되었는가 (또는 마스크 채널이 있는가)

## 검증 도구

```bash
ros2 run aidin_rby1_teleop validate_episode \
    --hdf5 /data/hdf5/pick_red_block_0001.hdf5
```

출력 예:

```
✓ duration: 12.3s (T=369 @ 30Hz)
✓ image_left:   369 frames, drop ratio 0.0%
✓ image_right:  369 frames, drop ratio 0.0%
✓ joint sync max gap: 6.8 ms (< 10 ms)
✗ NaN in action.wrist_left_target at t=4.21s (1 sample)  ← FAIL
```

## 시각화

- `tools/replay_episode.py` 로 HDF5 를 RViz/이미지 뷰어에서 재생
- 액션-관측 정합을 시각적으로 검토

## 데이터셋 인덱싱

검증을 통과한 에피소드만 `dataset_index.csv` 에 등록:

```csv
episode_id,path,task,operator,duration_s,success
pick_red_block_0001,/data/hdf5/pick_red_block_0001.hdf5,pick_red_block,jm,12.3,true
```

학습 단계는 이 인덱스 파일을 입력으로 받습니다.
