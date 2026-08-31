RTMP 프로토콜이 대중적인 송출 프로토콜로 사용되는 가운데, 
SRT 프로토콜이 무엇인지도 알아보자.


### SRT 프로토콜이란 ? 
SRT (Secure Reliable Transport) UDP 전송 프로토콜 
SRT는 TCP와 유사한 연결 및 제어, 안정적인 전송을 제공한다.

SRT는 TCP와 유사한 연결 및 제어 안정적인 전송을 제공한다. 
그러나, UDP 프로토콜을 기본 전송 계층으로 사용하여 어플리케이션 계층에서 이를 수행한다. 낮은 대기시간(기본 값:120ms)을 유지하며 패킷 복구를 지원한다. 
AES 같은 암호화도 지원함.

SRT는 라이브스트리밍을 지원하기 위해 몇가지 기능을 추가했다. 
1. 소스 시간 전송
2. 완화된 발신자 속도 제어 
3. 조건부 형태의 "너무 늦은" 패킷 삭제 (제떄에 복구되지 않은 손실된 패킷으로 인해 발생하는 HOL 차단 방지)
4. 즉시 패킷 재전송 (주기적인 NAK 보고)


### 데이터 구조
SRT 패킷은 UDP 페이로드로 전송됩니다. SRT 트래픽 전송하는 모든 UDP 패킷은 UDP 헤더 바로 뒤에 SRT 헤더를 포함합니다.
![](https://velog.velcdn.com/images/ohjiwoo123/post/ec3ee163-03e0-43a6-8629-21752d196e78/image.png)  

SRT 패킷은 아래와 같습니다.
![](https://velog.velcdn.com/images/ohjiwoo123/post/d187d6f9-71f1-4a2a-9a43-25727fc826ec/image.png)
   
- F: 1비트, 패킷 타입 플래그로 control packet = 1, data packet = 0으로 설정된다.
- Timestamp (32bits): ms 단위의 타임스탬프 필드이다.
- Destination SRT Socket ID(32bits) : 패킷을 전송할 SRT 소켓 ID를 제공하는 고정 너비 필드입니다. 

#### Data Packets
![](https://velog.velcdn.com/images/ohjiwoo123/post/4b767c87-bb53-4afd-8a78-cd424cda21ab/image.png)
- Packet Sequence Number (31 bits)
- PP (2bits) : Packet Position Flag. 10b는 첫 패킷의 메세지를 의미한다. 00b는 패킷의 중간을 의미한다. 01b는 마지막 패킷을 의미한다.  
- O (1bit) : Order Flag. Order (1) not (0) 순서대로 전송할지 아닐지
- KK (2bits) : Key-based Encryption Flag. 00b는 암호화 x, 01b는 even key와 암호화됨. 10b는 oddkey 암호화. 11b 는 Control Packets에서만 사용함.
- R (1bit) : Retransmitted Packet Flag. 재전송시 값이 1로 설정된다. 
- Message Number (26bits)
- Timestamp (32bits)
- Destination SRT Socket ID (32bits)
- Payload
- Authentication Tag (128bits)


#### Control Packets
![](https://velog.velcdn.com/images/ohjiwoo123/post/dfde2c4f-262f-4902-abd4-aeda30c30548/image.png)

Control Type (15bits)
Subtype (16bits)
Type-specific Information (32bits)
Timestamp (32bits)
Destination SRT Socket ID (32bits)
Control Information Field (CIF)

|Packet Type|Control Type|Subtype|Section|
|---|:---|:---:|---:|
|HANDSHAKE|0x0000|0x0|Section 3.2.1|
|KEEPALIVE|0x0001|0x0|Section 3.2.1|
|ACK|0x0002|0x0|Section 3.2.1|
|NAK (Loss Report)|0x0003|0x0|Section 3.2.1|
|Congestion Warning|0x0004|0x0|Section 3.2.1|
|SHUTDOWN|0x0005|0x0|Section 3.2.1|
|ACKACK|0x0006|0x0|Section 3.2.1|
|DROPREQ|0x0007|0x0|Section 3.2.1|
|PEERERROR|0x0008|0x0|Section 3.2.1|
|User-Defined Type|0x7FFF|-|N/A|


### SRT 서버 실행방법

SRT 빌드 후에 

```
srt-live-transmit <input-uri> <output-uri> [options]
```


```
# 리눅스서버에서
srt-live-transmit udp://:1234 srt://:4201 -v
```

```
# vlc player, 윈도우에서
srt://1.234.55.245:4201
```

```
# udp srt 4201 포트 tcpdump 하기
sudo tcpdump -i any udp port 4201 -w srt_4201.pcap
```

https://github.com/Haivision/srt/blob/master/docs/apps/srt-live-transmit.md


