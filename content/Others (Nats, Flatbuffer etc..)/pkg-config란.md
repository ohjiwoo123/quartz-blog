`pkg-config`는 라이브러리와 프로그램의 컴파일 및 링크 시 필요한 정보를 제공하는 도구이다.

리눅스에서 주로 사용되는 도구이다.   

pkg-config는 .pc 파일(패키지구성파일)에 정보를 저장한다.   
이파일은 /usr/lib/pkgconfig/ 또는 /usr/share/pkgconfig/ 에 위치한다.  


## 패키지 정보 확인 
`pkg-config --list-all`
다음 명령어는 시스템에서 사용할 수 있는 모든 패키지를 나열 한다.  


## 컴파일 플래그 확인 
특정 라이브러리를 컴파일에 사용할 때 필요한 플래그를 확인하려면 다음 명령을 사용합니다   
```
pkg-config --cflags 패키지이름 
ex)  
pkg-config --cflags glib-2.0

output  
-I/usr/include/glib-2.0 -I/usr/lib/glib-2.0/include
```

## 링크 플래그 확인 
링크 시 필요한 플래그를 확인하려면 다음 명령을 사용합니다   
```
pkg-config --libs 패키지이름
pkg-config --libs glib-2.0

output : 
-L/usr/lib -lglib-2.0
```

## 컴파일시 pkg-config 사용  

```
gcc main.c $(pkg-config --cflags --libs glib-2.0) -o main
== 
gcc main.c -I/usr/include/glib-2.0 -I/usr/lib/glib-2.0/include -L/usr/lib -lglib-2.0 -o main

```

## pc 파일 예제 
```
prefix=/usr
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: glib-2.0
Description: GLib is a general-purpose utility library
Version: 2.68.0
Libs: -L${libdir} -lglib-2.0
Cflags: -I${includedir}/glib-2.0 -I${libdir}/glib-2.0/include
```

## CMakeLists.txt에서 pkg-config 이용  

https://stackoverflow.com/questions/29191855/what-is-the-proper-way-to-use-pkg-config-from-cmake