BOOST는 C++ 라이브러리이다.  
굉장히 다양한 모델을 제공하는 라이브러리이다.  

### boost io_centext란 
io_context란 operating system과 I/O Object (소켓 등)의 연결 다리 입니다.  
작업영역에 대해서 내부적으로 큐로 관리합니다.

### boost에서의 동기 연결 
![](https://velog.velcdn.com/images/ohjiwoo123/post/e1d2ed73-8dad-4dcd-b386-99c905ee8d77/image.png)
  
- 최소 하나의 I/O execution context를 가져야 한다. 이 io_context는 OS와의 I/O services를 대표한다. 
`boost::asio::io_context io_context;
`
- I/O operations을 수행하기 위해서 I/O object가 필요하다 ex) TCP Socket
`boost::asio::ip::tcp::socket socket(io_context);
`
1. 연결 초기화가 필요하다 (I/O object)
`socket.connect(server_endpoint);
`
2. I/O 객체(소켓)는 요청을 I/O 실행 컨텍스트로 전달합니다.

3. io_context는 운영시스템을 호출합니다. 연결된 connect operation을 수행하기 위해서.

4. Operating system은 io_context의 결과 값을 반환합니다.

5. io_context는 에러가 발생하면 ` boost::system::error_code`를 통해 에러를 반환 합니다. 

6. I/O Object는 operation(수행)이 실패한 경우 에러를 반환합니다. 
```
boost::system::error_code ec;
socket.connect(server_endpoint, ec)
```
  

### boost에서의 비동기 연결 
![](https://velog.velcdn.com/images/ohjiwoo123/post/87d41784-2c7b-4e2c-9fc8-339bf76e371d/image.png)

1. I/O Object를 호출하므로서, connect operation을 초기화합니다.
`socket.async_connect(server_endpoint, your_completion_handler);`

* completion_handler는 아래와 같습니다.
`void your_completion_handler(const boost::system::error_code& ec);`

2. I/O 객체(소켓)는 요청을 I/O 실행 컨텍스트로 전달합니다.

3. io_context는 OS에 신호를 전달합니다 (비동기 연결이 시작되었다.)
동기 연결시에는 이 시간을 온전히 기다려야 했습니다.

![](https://velog.velcdn.com/images/ohjiwoo123/post/94d0fa09-46c4-43c2-8bf3-cf0e15489207/image.png)

4. OS(Operating System)는 연결에 대한 connect operation 완료했음을 나타내기 위해 큐에 넣고 io_context를 가져올 준비를 합니다. 

5. io_context를 사용할 때, io_context::run()을 반드시 사용해야 합니다.
(io_context::run() 함수 호출은 완료되지 않은 비동기 작업이 있는 동안 차단되므로, 일반적으로 첫 번째 비동기 작업을 시작하자마자 호출합니다.)

6. io_context::run()이 실행되는 동안, io_context는 큐에서 result of operation을 꺼내고, error_code로 반환함과 동시에, `completion_handler`로 전달합니다.


### io_context 모델 설계 방법  
io_context 는 다음의 네가지 모델로 분류된다.
1. 단일스레드에서의 단일 io_context 1개
2. 복수 스레드에서 io_context를 공유한다.
3. 쓰레드 당 io_context를 마련한다.
4. 복수 쓰레드에서 복수의 io_context를 사용한다. 



### 단일 쓰레드
`race condition`을 고려할 필요 없다.  
io_context 생성자의 concurrency_hint를 1로 지정함으로써 단일 스레드에서는 불필요한 배타 처리를 줄이고 성능을 조금이라도 올릴 수 있다.

이 모델에서 주의할 것은 각 핸들러의 실행 시간을 짧게 하는 것이다. 무거운 처리는 워커 쓰레드를 준비하고 그곳에서 실행한다.

```
void handler()
{
    auto work = asio::io_service::work{io_service};
    // work を保持させる
    worker_thread.post([work]{
        long_running_task(); // 何か時間がかかる処理
        work.get_io_service().post(next_task);
    });

    // io_service のキューが空でも work が存在するので
    // io_service::run は終了せず次のハンドラ (next_task) を待ち続ける.
}
```

### 복수 쓰레드, 1개의 io_context
이 모델에서 IO_CONTEXT는 1개이고 쓰레드풀에 있는 쓰레드가 호출된다.
```
asio::io_service io_service{};
std::vector<std::thread> thread_pool{};
for (auto i = std::size_t{0}; i < nthreads; ++i) {
    thread_pool.emplace_back([&io_service]{
        io_service.run(); // invoke run for each thread
    });
}
```
한편, 기본적으로 Asio에서 제공되는 I/O Object는 스레드 세이프 하지 않으므로 Strand 또는 Mutex를 사용하여 스레드 간 동기를 해야할 필요가 있다.


### 쓰레드 당 io_context
이 모델에서는 스레드마다 한개의 io_service가 존재한다.

```
std::vector<asio::io_service> io_service_pool(nio_services);
std::vector<std::thread> threads{};

for (auto& io_service : io_service_pool) {
    threads.emplace_back[&io_service]{ // bind each io_service
        io_service.run();
    }
}
```
이 모델의 경우 각 I/O Object는 하나의 스레드에 속하게 되므로 기본적으로 Strand 등을 사용하여 동기 할 필요가 없다.
단일 스레드 모델의 설명에서 이야기한 concurrency_hint도 적용할 수 있으며, io_service의 수(쓰레드 수)를 CPU 수에 맞춰지면 퍼포먼스적으로는 스레드 풀 모델보다 좋을 것이다.
한편, 단일 스레드 모델과 마찬가지로 핸들러의 실행 시간에 주의할 필요가 있다.

### 복수 쓰레드에서 복수 io_context
- 생략하겠다.   


지금 하려는 건 코어당 3번의 주제이다.  
쓰레드당 io_context에 해당하는 것 같다.   



참고 사이트 :

그림 예제
https://think-async.com/Asio/boost_asio_1_30_2/doc/html/boost_asio/overview/basics.html

한글로 잘 정리된 사이트 
https://chelseafandev.github.io/2021/12/28/boost-io-context/  

일본어로 된 io_context 모델 정리   
https://github.com/jacking75/boost_asio_sample/blob/master/Document/asio_thread_model.md
