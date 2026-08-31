## LL-HLS (Low-Latency HLS) 조사  
일반적인 HLS 는 긴 LATENCY를 가진다.  
일반적인 서버-클라이언트 구조에서 6초의 세그먼트를 전달한다고 가정할 때, 첫 프레임을 받기 까지 최소 12초가 걸린다. 그리고 일반적으로 많은 플랫폼들이 CDN을 가지니 CDN 거치는 시간을 고려하면 시간이 더 지연된다.   

그리하여 탄생하게 된 것이 `Low-Latency HLS` 이다.

### Partial Segments

### Playlist Delta Updates 
- `EXT-X-SERVER-CONTROL` with `CAN-SKIP-UNTIL` attribute
EXT-X-SERVER-CONTROL (CAN-SKIP-UNTIL 속성이 있는 사용한다.) 
- 클라이언트는 델타 업데이트를 명시적으로 요청한다.
- 업데이트는 이미 클라이언트가 가지고 있는 플레이리스트의 이전 부분을 건너뜁니다.

GET https://example.com/1M/live.m3u8?_HLS_skip=YES

```
GET https://example.com/1M/live.m3u8?_HLS_skip=YES

#EXTM3U
#EXT-X-VERSION:9
#EXT-X-SERVER-CONTROL:CAN-SKIP-UNTIL=36.0 // 라이브 엣지 직전 36초까지 모든 것을 건너뜁니다.
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:100
#EXT-X-SKIP:SKIPPED-SEGMENTS=1700  // 1700번대 세그먼트들은 스킵한다
#EXTINF 6.0,
segment1800.ts
#EXTINF 6.0,
segment1801.ts
#EXTINF 6.0,
segment1802.ts
```

### Blocking Playlist Reload
- `EXT-X-SERVER-CONTROL` with `CAN-BLOCK-RELOAD` attribute
EXT-X-SERVER-CONTROL (CAN-BLOCK-RELOAD 속성이 있는 사용한다.) 

- 클라이언트는 다음 플레이리스트를 미리 요청한다.
서버는 요청을 가지고 있는다 다음 세그먼트 또는 부분 세그먼트가 나타날때까지.. 


- 클라이언트는 서버에 어떻게 플레이리스트를 요청하는가?  
--> `GET https://example.com/live.m3u8?_HLS_msn=1803`   
(미디어시퀀스 1803 에 해당하는 플레이리스트를 달라!)   

--> `GET https://example.com/live.m3u8?_HLS_msn=1803&HLS_part=0&_HLS_push=1`
(1803 미디어시퀀스의 첫번째 세그먼트를 달라!)  

HLS_push = 서버에게 HTTP/2 Push 또는 Chunked Transfer Encoding으로 다음 part를 미리 보내달라는 요청  


![](https://velog.velcdn.com/images/ohjiwoo123/post/04ef621d-4412-4308-92e2-0239e51aba8e/image.png)


### Rendition Reports
- 재생 목록 업데이트는 피어 재생 목록에 대한 최신 보고서를 포함할 수 있습니다.

- 마지막 시퀀스 번호와 마지막 부분 세그먼트 번호
- 최신 플레이리스트를 load하는 것을 허용한다 비트레이트가 변경될 때

```
Requesting and receiving Rendition Reports

GET https://example.com/1M/live.m3u8?_HLS_REPORT=/2M/live.m3u8
#EXTM3U
#EXT-X-RENDITION-REPORT:URI="/2M/live.m3u8",LAST-MSN=1801,LAST-PART=0
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:1800
#EXTINF 6.0,
segment73.ts
```


## Configure Your CDN

Deliver HLS via HTTP/2
- Support HTTP/2 Push
- Support HTTP/2 dependency and weighting

Each server must vend all bitrate tiers  

CDN must aggregate duplicate pending rquests to origin
(CDN은 중복된 보류 중인 요청을 원본으로 집계해야 합니다.)  


## Reference Implementation
"Low-Latency HLS Beta Tools"