### REDIS란 ? 
Remote Dictionary Server의 약자로, 다수의 서버가 공유하는 해시 테이블이다.

- Remote : Redis 서버는 각각의 서버안에 로컬하게 존재하는 것이 아니라, 개별적인 원격상으로 존재하여 다수의 서버가 공통으로 사용할 수 있다.
- Dictionary : Key-Value 쌍으로, 상수의 시간복잡도로 사용 가능하다.

### REDIS 특징 
1. 레디스는 표준 C로 작성 된 오픈소스 인메모리 데이터 저장소이다. 
2. 백업을 제외한 모든 데이터를 램에 저장하기 때문에 RDBMS와 다른 구조를 가지며, 인메모리 데이터 스토어다. (일반적으로 램은 디스크보다 빠르다.)
3. **싱글 스레드** 기반 이다. 개발자가 잘 활용하면 사이드 이펙트가 거의 없다.
4. 클러스터 모드를 지원한다. 다중 노드에 데이터를 분산 저장하여 기본적으로 안정성-고가용성을 제공한다.
5. 인메모리 데이터베이스라는 특성상 주로 휘발성 데이터를 저장하지만, RDB와 AOF(Append Only File)이라는 특성을 통해 안전하게 영속적으로 관리할 수도 있다.
6. Pub/Sub 같은 기술이 자체적으로 구현되어 있어서, 이를 활용하여 실시간 채팅이나 알림 서비스 어플을 손쉽게 개발할 수 있다. 

### 레디스 장점
1. 모든 데이터를 메모리에 저장하기 때문에 매우 빠른 읽기/쓰기 속도를 보장한다.
2. 다양한 데이터 타입을 지원한다.
3. 다양한 언어로 작성된 클라이언트 라이브러리를 지원한다. (백엔드 연동이 쉽다.)

### 레디스 사용 사례
1. Caching : 임시 비밀번호/로그인 세션과 같은 임시 데이터를 레디스에 캐싱하여 활용하는 사례가 많다.
2. Rate Limiter (Fixed-Window/Sliding-Window Rate Limiter) : 서버에서 특정 API에 대한 요청 횟수를 제한할 때 사용한다.
3. MESSAGE BROKER : 레디스의 리스트나 스트림지 같은 데이터 타입을 활용하여 메세지 브로커를 구현할 수 있다. 다양한 서비스 간의 커플링을 줄일 수 있다.
4. 실시간 분석/계산 : 다양한 실시간 분석 및 계산이 가능하다.
5. 실시간 채팅 : 레디스의 Pub/Sub 패턴을 활용하여 실시간 채팅 기능을 구현할 수 있다.

### 레디스 영속성
- 레디스는 주로 캐시로 사용되지만, 데이터 영속성을 위한 옵션을 제공한다. SSD와 같은 영구적인 저장 장치에 데이터를 저장하는 방식으로 구현된다.

1. RDB (Redis DataBase)
- 특정 시간에 스냅샷을 생성하는 기술이다. 장애 발생시, 빠르게 특정 시점의 스냅샷으로 캐시를 되롤리거나 동일한 데이터를 가진 캐시를 복제할 때 주로 사용한다.
- 스냅샷 트성상 일부 데이터 유실 위험이 있다.
- 스냅샷 생성중에 전체적인 레디스 서버 성능 저하가 발생하여 클라이언트 요청 처리에 지연이 발생할 수 있다.

2. AOF (Append Only File)
- 레디스에 적용되는 Write 작업을 모두 log로 저장하는 방식이다.
- 데이터 유실 없이 모든 데이터 싱크를 맞출 수 있지만, 장애상황 재난 복구 시에 모든 로그를 다시 적용해야해서, 스냅샷 방식보다 복구속도가 느리다.

### Caching 
1. CPU 캐시 : CPU와 RAM 속도 차이로 발생하는 지연을 줄이기 위해 캐시 사용
2. 웹브라우저 캐싱 : 웹 브라우저가 웹 페이지 데이터를 로컬 저장소에 저장하여 해당 페이지 재방문 시 사용 
3. DNS 캐싱 : 이전에 조회한 도메인 이름과 해당 IP 주소를 저장하여 재요청 시 사용
4. DB 캐싱 : DB조회나 계산 결과를 저장하여 재요청시  사용
5. CDN : 원본 서버의 컨텐츠를 PoP 서버에 저장하여, 사용자와 가까운 서버에서 요청처리 (이미지/동영상 등 용량이 큰 파일을 PoP 서버에 저장해두고, 사용자가 요청을 보냈을 때, 가장 가까운 PoP 서버에 있는 파일을 응답 값으로 보내줌으로써 네트워크 지연시간을 줄임.)
6. 어플리케이션 캐싱 : 어플리케이션에서 데이터나 계산 결과를 캐싱하여 반복 작업 최적화

### How To Install 
Rocky Linux 개발서버가 있으므로, 아래 공식홈페이지의 스텝을 따라서 설치한다.

https://redis.io/docs/latest/operate/oss_and_stack/install/install-stack/rpm/




### Client API
C/C++ 유저로서, C 라이브러리인 hiredis를 사용한다. 

https://github.com/redis/hiredis  

정적라이브러리 파일은 빌드되지 않으며, 동적라이브러리 파일을 사용한다. 
--> .so 파일  
그러므로 CMakeLists.txt 같은 것을 작성할 때는, 
```
target_link_libraries(
    ${tname}
    ${CMAKE_DL_LIBS}
    hiredis dl
```

**동적라이브러리 링킹에 관한 글  
출처 : https://blog.naver.com/lifeisforu/222665130680

```
# include <stdio.h>
# include <stdlib.h>
# include <hiredis.h>

int main(void){
  redisContext *conn = NULL;
  redisReply *resp   = NULL;
  int loop           = 0;
  
  // 接続
  conn = redisConnect( "127.0.0.1" ,  // 接続先redisサーバ
                       6379           // ポート番号
         );
  if( ( NULL != conn ) && conn->err ){
    // error
    printf("error : %s\n" , conn->errstr );
    redisFree( conn );
    exit(-1);
  }else if( NULL == conn ){
    exit(-1);
  }
  
  // Valueの取得
  for( loop=1 ; loop < 4 ; loop++ ){
    resp = (redisReply *) redisCommand( conn ,          // コネクション
                                        "GET %d" , loop // コマンド
                                      );
    if( NULL == resp ){
      // error
      redisFree( conn );
      exit(-1);
    }
    if( REDIS_REPLY_ERROR == resp->type ){
      // error
      redisFree( conn );
      freeReplyObject( resp );
      exit(-1);
    }
    printf( "%d : %s\n" , loop , resp->str );
    freeReplyObject( resp );
  }
  
  // 後片づけ
  redisFree( conn );
  return 0;
}

gcc -Wall -o redis redis.c -lhiredis -L/usr/local/lib -I/usr/local/include/hiredis

./redis
1 : aaa
2 : bbb
3 : ccc

출처 : https://qiita.com/Ki4mTaria/items/d73cf3d244c903d493eb
```
