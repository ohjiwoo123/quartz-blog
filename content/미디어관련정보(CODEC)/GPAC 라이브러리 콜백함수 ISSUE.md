## 문제점  
GPAC 콜백함수에 이슈가 있습니다.  

첫번째 FMP4 파일이 STYP MOOF MDAT의 형태로 저장되지 않아서   
VOD 저장 시 트리의 형태가 무너지는 이슈입니다.  

지팩 콜백으로 SIDX 옵션을 OFF 하는 경우   
두가지의 형태로 박스가 오는 것 같다.  

CASE 1    
STYP / MOOF 
MDAT 의 형태로 오느냐

CASE 2  
STYP / MOOF 
MDAT / MOOF 

결론은 sidx 옵션을 껐을때  
`gf_bs_prevent_dispatch` 이녀석으로 인해   
콜백이 2번 불리는것 같다.  

정확히는   
`gf_isom_close_segment`  함수 안에
`StoreFragment`에 있는 `gf_bs_prevent_dispatch` 함수를 주석하느냐 안하느냐의 차이로  
지팩 내부에서 콜백이 2번 불리는 것이다.  
하지만 `gf_bs_prevent_dispatch` 함수를 주석 처리하면 해결 될 것 같지만,  
정확하게 이 함수가 어떤 함수이고, 어떤 영향을 미칠 수 있는지 조사가 필요하다.  

https://github.com/gpac/gpac/blob/7b6b49900a54ae310e474c6244aba90bc20c95f6/src/isomedia/movie_fragments.c#L1251    

## gf_bs_prevent_dispatch
함수에 대한 설명  
https://doxygen.gpac.io/group__bs__grp.html#ga9ab1ea6baed1879a68be6ea48011fe13   

간단하게 요약하면,   
`Block dispatch prevention acts in a counter mode: you must call as many time the function with prevent_dispatch = GF_FALSE as you called the function with prevent_dispatch = GF_TRUE
`
chat-gpt에게 한글로 설명을 부탁해서 아래와 같은 답변을 얻었다.  

`gf_bs_prevent_dispatch()` 함수는 특정 비트스트림(bitstream) 객체에서 데이터를 일시적으로 내부에 저장하게 하거나, 저장된 데이터를 다시 외부로 전송하게 하는 기능을 제어하는 함수입니다.

이 함수는 prevent_dispatch라는 매개변수를 통해 비트스트림의 데이터 전송을 차단하거나 허용할 수 있습니다. prevent_dispatch가 GF_TRUE로 설정되면 비트스트림의 데이터가 외부로 즉시 전송되지 않고 내부적으로 임시 저장소에 저장됩니다. 반대로 prevent_dispatch가 GF_FALSE로 설정되면, 이전에 내부적으로 저장된 데이터를 다시 외부로 전송할 수 있게 됩니다.

이 함수는 "카운터 모드"로 동작하므로, prevent_dispatch를 GF_TRUE로 설정한 횟수만큼 GF_FALSE로 설정하는 호출을 해야 비트스트림의 데이터 전송이 다시 정상적으로 이루어집니다. 즉, 일시적으로 데이터를 저장하는 상태와 정상적으로 데이터를 전송하는 상태를 전환하는 데 사용되는 함수입니다.  

이함수 사용 이전엔 다음과 같은 주석이 있다.  
/*we don't want to dispatch any block until done writing the moof*/   
moof 박스를 저장하는 동안 dispatch 되는 것을 원하지 않는다.  

내 개인적인 생각엔 moof 를 작성하는 동안 mdat의 데이터가 침범할 수 있어서  
이런 장치를 마련 해놓은 것 같다.  
이말은 즉슨, gpac 내부 라이브러리를 고쳐서 해결하려하기 보다는, 우리가   
콜백함수안에서 제어해서 데이터를 컨트롤해야하는 것이 아닌가 하는 생각이 든다.  
이부분은 오픈소스인 만큼, 이슈로 기록을 남겨서 함수작성자의 의도를 물어보는 것도 좋을 듯 하다.  

질문 올리고 내용 이어서 기록 해야 겠다.  

gpac 오픈소스 라이브러리에 질문 올린 내용  
https://github.com/gpac/gpac/issues/2946   

++ 추가 답변은 받았고,
해당 부분은 우리가 내부 버퍼컨트롤을 하여 해결하였음. 