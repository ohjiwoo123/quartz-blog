비디오 오디오 모두 빠른 개발을 위하여 fixed 된 데이터를 편의상으로 사용하였는데,
전부 OBS 외부장치에서 오는 데이터를 바탕으로 데이터를 파싱 및 할당하여 문제 해결 됨.
RFC6381

오디오의 경우
`Audio Specific Config raw data`를 앞단 (RTMP,미러)쪽에서 보내주지 않고 있어서
통합 스트림에서 임의로 Sampling rate와 channel, Audio Object Type을 바탕으로
`Audio Specific Config raw data`를 역으로 계산해서 만들어서 사용하였습니다.

objectTypeIndication = 0x67    GF_CODECID_AAC_MPEG2_LCP  
objectTypeIndication = 0x40    GF_CODECID_AAC_MPEG4   


현재는 objectTypeIndication = 0x40    GF_CODECID_AAC_MPEG4
이렇게 되면 codec id = mp4a.40.2
이걸 사용하지만
나중에 기준에 따라서 정확히 할당해서 사용하는 것이 나을 것 같음
비디오의 경우
OBUS(시퀀스헤더)를 풀파싱하여 AV1_CONFIG 값을 할당해주었으며 추가로,
RFC 6381에는 아래와 같은 추가로 필요합니다.

```
/*Needed for RFC6381*/
Bool still_picture;
u8 bit_depth;
Bool color_description_present_flag;
u8 color_primaries, transfer_characteristics, matrix_coefficients;
Bool color_range;
```

따라서 AV1 CONFIG 할당 추가해주고,
COLOR에 해당하는 변수들도 추가해주고,
해상도 옵션도 추가해줍니다.

gf_isom_av1_config_new
gf_isom_set_visual_info
gf_isom_set_visual_color_info


그리고 비디오의 코덱 네임은 RFC 6381에서 코덱정보를 알아내는 중요한 정보이므로
정확한 값을 할당해주어야 합니다.
av01.P.LLT.DD[.M.CCC.cp.tc.mc.F]
https://developer.mozilla.org/en-US/docs/Web/Media/Formats/codecs_parameter#av1
OBUS 파싱한 결과로 임의로 만들어 줄 수도 있습니다.
하지만 gpac에서 제공하는 api가 정확한 함수를 제공해주므로 (문서와 비교하여 확인완료) 그대로 사용해줍니다.
gf_media_get_rfc_6381_codec_name

결론적으로 VLC PLAYER와 FFPLAY 같은 외부 프로그램을 사용하면
완전한 스트림을 넣지 않더라도 재생이 되지만 (ex ) 송출되는 것과 다른 코덱정보를 넣는경우), HLS의 경우 엄격한 검사를 진행하기 때문에
모든 데이터를 올바르게 받아서 할당해주는 것으로 진행하겠습니다.
==> 우선 코덱 정보 이슈는 현재 해결되었습니다.