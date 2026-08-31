
.dll은 윈도우에서 사용하는 동적라이브러리 파일이고, 
.so는 리눅스에서 사용하는 동적라이브러리 파일이다.  


아래의 링크에서 윈도우 dll 설명 및 사용법에 대해 잘 나와 있다.  

출처 : https://wnsgml972.github.io/setting/2018/11/01/dll_lib/  


## 리눅스 so
리눅스는 아래와 같은 step을 따른다.  

hello.c
```
#include <stdio.h>

void hello()
{
	printf("Hello World\n");
}
```

```
gcc -c -fPIC hello.c -o hello.o
gcc hello.o -shared -o libhello.so 
or 
gcc -shared -o libhello.so -fPIC hello.c
```
위와 같이 명령어를 사용하면 `libhello.so`라는 동적라이브러리 파일이 만들어진다.  

동적라이브러리 파일을 사용해보고 싶다면,  

main.c
```
// 외부에서 제공되는 함수 선언
void hello();


int main()
{
hello(); // libhello.so
return 0;
}

```

```
# 라아브러리 경로 추가  
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
# 컴파일 
gcc main.c -L. -lhello -o main

./main 실행  
output : Hello World
```
출처 : https://stackoverflow.com/questions/14884126/build-so-file-from-c-file-using-gcc-command-line  


## CMakeLists.txt 동적라이브러리 추가 방법   
일반적인 프로젝트를 빌드할 떄, 일일이 
`gcc main.c -L. -lhello -o main`  
명령어를 사용하여 빌드하지 않고, 편의를 위해서 CMakeLists.txt를 활용하여 Makefile을 생성 후 빌드하는 것을 선호한다.  

따라서 이 방법도 알아보자.  


라이브러리 만들기
```
add_library(my_shared_object  SHARED  # 동적 링킹 라이브러리(.so, .dll)
    main.cpp
    feature1.cpp
    # ...
)
```

링킹하는건 static 처럼 똑같은듯  




