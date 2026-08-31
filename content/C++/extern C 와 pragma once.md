
## 1. extern "C" 를 사용하는 이유는 ? 
```
#ifdef __cplusplus
extern "C" {
#endif

// C 함수 선언들

#ifdef __cplusplus
}
#endif
```

--> C 방식으로 컴파일해 
C++은 (Name Mangling)으로 함수 이름에 타입정보를 붙여 컴파일 한다. [오버로딩 특성으로 인해]
ex) 
_Z3foov
_Z3fooi

이로 인해 링크 에러가 발생하므로 extern "C" 를 사용하면 좋다. 


## 2. #pragma once
헤더 파일이 한 번만 포함되도록(included only once) 하는 전처리기 지시문

예전 방식 (Include Guard)
```
#ifndef A_H
#define A_H
#endif 
```

요즘은 #pragma once를 많이 사용한다고들 한다.