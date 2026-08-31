## sps로부터 fps 계산하는법

### frame_rate_flag = 0 인 경우

You will need to decode PTS/DTS associated with each picture (if they are present) to compute the average frame rate.

평균적인 프레임 dts pts 분석을 통해, fps를 직접 구해야한다. 

### frame_rate_flag = 0 이 아닌 경우
In general, 
`time_scale / (2 * num_units_in_tick)`

vui_prameters_present_flag , timing_info_present_flag 둘다 1 이면,
fps를 구할 때 필요한 값은 
num_units_in_tick, time_scale 이다.

fixed_frame_rate_flag가 1이면,
일반적으로 fps 는 time_scale / num_units_in_tick 이다.

만약 field-based video면 , 반으로 나누면 된다. 

field-based 인지 frame-based 인지는 
frame_mbs_only_flag로 판별하는데 0이면 필드기반, 
1이면 프레임 기반 이다.

[출처]
https://stackoverflow.com/questions/31334973/find-frame-rate-sps
https://forum.doom9.org/archive/index.php/t-126584.html

### ffprobe로 fps 확인해보는 방법
```
ffprobe -v error -select_streams v:0 \
-show_entries stream=avg_frame_rate,r_frame_rate \
-of default=noprint_wrappers=1 shark.h264
r_frame_rate=60000/1001
avg_frame_rate=25/1

output : 
r_frame_rate=60000/1001
avg_frame_rate=25/1

```


## B프레임과 P프레임  

1. GOP 단위로 모은다.
2. POC (Picture of Order Count 기준으로 정렬한다.) 
3. 정렬된 순서대로 PTS 증가
`pts = base_pts + display_index * video_step;`


POC 는 slice_header에 들어 있다.
```
pic_order_cnt_lsb
delta_pic_order_cnt_bottom
delta_pic_order_cnt[0/1]
```
이 값들은 SPS의 `pic_order_cnt_type`에 따라 해석이 달라진다.

|type|설명|실무|
|---|:---|:---|
|0|`pic_order_cnt_lsb` 기반|가장 흔함|
|1|delta 기반 누적|드묾|
|2|decode order 기반|B-frame 없음|

pic_order_cnt_type == 0 인 경우, 99% 
pic_order_cnt_lsb  (예: 0, 2, 4, 6 ...)
log2_max_pic_order_cnt_lsb_minus4

```
MaxPOCLsb = 1 << (log2_max_pic_order_cnt_lsb_minus4 + 4);

POC = prevPOC + delta; // wrap-around 처리 포함
```

### num_ref_frames 가 큰 경우 재생이 안되는 이슈

스마트티비에서 sps의 num_ref_frames가 8 이렇게 찍히면 재생이 안되는 이슈가 있음. 스마트티비의 플레이어인 티젠 등 OS에 따라서 num_ref_frames가 큰 경우 재생이 안됨.  


### 비디오 비트레이트 
비디오 비트레이트는 다음과 같이 구할 수 있다.  
1. RTMP에서 onMetaData AMF 패킷의 videodatarate 필드를 확인해야 함.  
2. 서버로 부터 들어오는 비디오 NAL Unit의 페이로드 크기 (Bytes)를 1~3초 단위의 윈도우로 누적 계산합니다.

```
# 비디오 비트레이트 구하는 공식 참고  Bitrate (kbps)
gap = ((current_time - m_nLastCheckTime) > 60초 보다 크면 
m_nBitrateTraffic += (tFrameInfo.uFrameSize * 8);
int32_t video_bitrate = ((m_nBitrateTraffic / gap) / 1000);
```

3. SPS Level을 통한 대략적인 유추 

### 비디오 해상도 
비디오 해상도의 경우 
RTMP 스펙의 OnMetaData의 필드에서도 확인 가능하고,
sps 에서 직접 파싱하여 구할 수도 있다. 
직접 파싱하는 경우에는, 

https://stackoverflow.com/questions/31919054/h264-getting-frame-height-and-width-from-sequence-parameter-set-sps-nal-unit

를 참고하자.

```
int SubWidthC;
int SubHeightC;

if (sps->chroma_format_idc == 0 && sps->separate_colour_plane_flag == 0) { //monochrome
    SubWidthC = SubHeightC = 0;
}
else if (sps->chroma_format_idc == 1 && sps->separate_colour_plane_flag == 0) { //4:2:0 
    SubWidthC = SubHeightC = 2;
}
else if (sps->chroma_format_idc == 2 && sps->separate_colour_plane_flag == 0) { //4:2:2 
    SubWidthC = 2;
    SubHeightC = 1;
}
else if (sps->chroma_format_idc == 3) { //4:4:4
    if (sps->separate_colour_plane_flag == 0) {
        SubWidthC = SubHeightC = 1;
    }
    else if (sps->separate_colour_plane_flag == 1) {
        SubWidthC = SubHeightC = 0;
    }
}

int PicWidthInMbs = sps->pic_width_in_mbs_minus1 + 1;

int PicHeightInMapUnits = sps->pic_height_in_map_units_minus1 + 1;
int FrameHeightInMbs = (2 - sps->frame_mbs_only_flag) * PicHeightInMapUnits;

int crop_left = 0;
int crop_right = 0;
int crop_top = 0;
int crop_bottom = 0;

if (sps->frame_cropping_flag) {
    crop_left = sps->frame_crop_left_offset;
    crop_right = sps->frame_crop_right_offset;
    crop_top = sps->frame_crop_top_offset;
    crop_bottom = sps->frame_crop_bottom_offset;
}

int width = PicWidthInMbs * 16 - SubWidthC * (crop_left + crop_right);
int height = FrameHeightInMbs * 16 - SubHeightC * (2 - sps->frame_mbs_only_flag) * (crop_top + crop_bottom);
```