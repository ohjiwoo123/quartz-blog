- h264 profiles은 크게 3가지가 있다. 
1. Base Profile
2. Main Profile
3. High Profile 

- profile_idc 
Baseline = 66
main = 77
high = 100 

### Base Profile
[Base Profile] 인 경우에 , 
1. I, P 프레임만 존재함. 
2. nal_unit_type 2~4 존재해서는 안 된다.
3. frame_mbs_only_flag equal 필드가 1로 설정되어야 한다.
4. chroma_format_idc
bit_depth_luma_minus8
bit_depth_chroma_minus8
qpprime_y_zero_transform_bypass_flag
seq_scaling_matrix_present_flag
가 sps에 존재해서는 안 된다.
5. pps의 weighted_pred_flag, weighted_bipred_idc가 모두 0이어야 한다.
6. pps의 entropy_coding_mode_flag가 0이어야 한다. 
(즉, CABAC을 사용할 수 없고 CAVLC만 사용해야 한다는 의미입니다.)
7. pps에서 num_slice_groups_minus1의 값은 0 이상 7 이하 여야 한다.
(실제 Slice Group 개수는 num_slice_groups_minus1 + 1이므로
1~8개의 Slice Group을 의미합니다.)
8. pps에는 transform_8x8_mode_flag pic_scaling_matrix_present_flag second_chroma_qp_index_offset 구문 요소가 포함되어서는 안 된다.
9. level_prefix 가 15를 초과해서는 안 된다.
10. PCM 샘플이 존재하는 경우 각 샘플 값은 0이어서는 안 된다.
11. A.3 절에 정의된 Baseline Profile의 Level 제약 조건을 모두 만족해야 한다.

### Main Profile 
1. I,P,B 슬라이스가 존재한다.
2. nal_unit_type 2~4 존재해서는 안 된다.
3. 임의의 슬라이스 순서는 허용되지 않는다.
4. chroma_format_idc
bit_depth_luma_minus8
bit_depth_chroma_minus8
qpprime_y_zero_transform_bypass_flag
seq_scaling_matrix_present_flag
가 sps에 존재해서는 안 된다.
5. pps의 num_slice_groups_minus1 가 0 이어야 한다.
6. pps의 redundant_pic_cnt_present_flag가 0 이어야 한다.
7. pps에는 transform_8x8_mode_flag pic_scaling_matrix_present_flag second_chroma_qp_index_offset 구문 요소가 포함되어서는 안 된다.
8. level_prefix 가 15를 초과해서는 안 된다.
9. pcm_sample_luma[i](i = 0..255)와 pcm_sample_chroma[i](i = 0..2 × MbWidthC × MbHeightC − 1) 구문 요소는 존재하는 경우 그 값이 0이어서는 안 된다.
10. 부록 A.3에서 정의된 Main Profile의 Level 제약 조건을 만족해야 한다.

### High Profile
1. I,P,B 슬라이스가 존재한다.
2. nal_unit_type 2~4 존재해서는 안 된다.
3. 임의의 슬라이스 순서는 허용되지 않는다.
4. pps의 num_slice_groups_minus1 가 0 이어야 한다.
5. pps의 redundant_pic_cnt_present_flag가 0 이어야 한다.
6. SPS(Sequence Parameter Set)의 chroma_format_idc는 0 또는 1만 사용할 수 있다.
7. SPS의 bit_depth_luma_minus8은 반드시 0이어야 한다.
8. SPS의 qpprime_y_zero_transform_bypass_flag는 반드시 0이어야 한다.
9. 부록 A.3에 정의된 High Profile의 Level 제약 사항을 만족해야 한다.