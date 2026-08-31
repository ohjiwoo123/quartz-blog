https://streaminglearningcenter.com/blogs/open-and-closed-gops-all-you-need-to-know.html  
위의 링크 바탕으로, 
IDR-FRAME, I-FRAME 심화 정리를 해보려고한다.   

## IDR 프레임이란  
- IDR 프레임은 H264의 특별한 I프레임이다.  
IDR 프레임 이후의 프레임이 이전 프레임을 참조할 수 없도록 지정합니다.  
중요한 것은 모든 IDR-FRMAE은 I프레임이지만  
반대의 경우는 아니다.  

X264 코덱은 기본적으로 CLOSED-GOP를 사용한다.  

## ABR Videos Should Use Closed GOPS
- adaptive bitrate video는 CLOSED-GOP를 사용해야 한다.  
예를들어 화질전환을 하는경우, 360p -> 1080p 세그먼트 시작의 I-frame 이전 프레임들이 인코딩중에 참조된 프레임들과 다르므로 문제가 발생할 수 있다.  

For HLS, the Apple HLS Authoring Specification reads, “Key frames (IDRs) SHOULD be present every two seconds.” So, all keyframes should be IDR frames, which means all GOPs are closed.
매 2초마다 IDR 프레임이 있어야하고, 이말은 GOP는 CLOSED-GOP를 사용해야 한단 말이다.  



## IDR-FRAME과 I-FRAME 구분 방법 
I프레임 여부는 slice_type 을 통해 확인해야하고, 
IDR프레임은 nal_unit_type == 5 이면 IDR-프레임이면서 I-프레임이다.  



![](https://velog.velcdn.com/images/ohjiwoo123/post/45b938ae-5904-475f-95dc-8afbad5f35ab/image.png) 
만약 IDR 프레임이면 slice_type은 2,4,7,9 여야만 한다. 
max_num_ref_frames == 0, slice_type은 2,4,7,9   

실제로는 2,7만 들어오지 않을까 싶음   