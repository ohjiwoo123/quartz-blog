### Nats 란 
- 오픈소스 메세징 시스템으로 서로의 네트워크 주소를 알지 못해도 subject를 기준으로 통신이 가능하다. 

- 작은 바이너리 파일, 높은 메세지 전송속도에서 낮은 지연 시간 안전한 환경을 제공합니다. 노트북의 단일 서버부터 여러 지역에 걸쳐 있는 서버 클러스터 까지 다양한 환경에서 실행 된다.


### 주요 Components
1. NATS SERVER : 메세지 브로커로서 메세지를 라우팅 
2. PUBLISHER : 메세지를 보내는 주체
3. SUBSCRIBERS : 메세지를 받는 주체 
4. SUBJECTS : 채널 이름


### 성능
- 초당 수백만개의 메세지 처리 가능 
- 밀리초 미만의 지연 시간
- 작은 메모리 사용량 (일반적으로 약 15MB)


### PUBLISH-SUBSCRIBE
1. PUBLISHERS가 서브젝트로 메세지를 보낸다.
2. SUBSCRIBERS가 그들이 구독하고 있는 서브젝트에 관심을 표현한다.
3. NATS가 모든 구독자에게 PUBLISHERS로 부터 받은 메세지를 전달한다.
4. PUBLISHERS는 구독자를 알지 못한다. SUSBSCRIBERS 또한 PUBLISHERS를 알지 못한다.

5.PUB/SUB PATTERN
FAN-OUT : 하나의 PUBLISHER 여러명의 SUBSCRIBERS
FAN-IN : 하나의 SUBSCRIBERS 여러명의 PUBLISHERS

6. 서브젝트 계층적 구분
```
orders.us.created
orders.eu.created
orders.us.canceled
```

### REQUEST-REPLY

1. 클라이언트가 유니크한 reply-subject를 만든다. (inbox라고 부름)
2. 클라이언트가 reply-subject를 구독한다. 
3. 클라이언트가 요청을 보낸다. reply-subject로 
4. 서비스가 요청을 받는다. reply-subject로 부터 
5. 서비스가 응답을 보낸다. reply-subject로  
6. 클라이언트가 응답을 받는다.


https://docs.nats.io/concepts/what-is-nats
https://docs.nats.io/concepts/request-reply