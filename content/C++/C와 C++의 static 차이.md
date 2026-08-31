[[메모리구조]]

#### C 에서의 STATIC
- C에서는 STATIC의 용도가 아래와 같다.
1. 범위 제한의 용도 (INTERNAL LINKAGE)
		1) 전역변수 static : 해당 .c 파일에서만 접근 
		2) 함수 static : 해당 .c 파일에서만 호출 
		3) 지역변수 static : 함수가 끝나도 값을 유지한다. 

#### C++에서의 STATIC
- C++에서는 STATIC의 용도가 아래와 같다.
1. 범위 제한의 용도  (INTERNAL LINKAGE)
		1) 전역변수 static : 해당 .c 파일에서만 접근 
		2) 함수 static : 해당 .c 파일에서만 호출 
		3) 지역변수 static : 함수가 끝나도 값을 유지한다. 
		4) 클래스멤버 static : 클래스 전체가 공유하는 변수/함수


CPP REFERENCE 내용
- [declarations of namespace members with static storage duration and internal linkage](https://en.cppreference.com/cpp/language/storage_duration "cpp/language/storage duration")
- [definitions of block scope variables with static storage duration and initialized once](https://en.cppreference.com/cpp/language/storage_duration#Static_block_variables "cpp/language/storage duration")
- [declarations of class members not bound to specific instances](https://en.cppreference.com/cpp/language/static "cpp/language/static")


C에서의 개념은 생략하고, C++에서의 개념을 조금 더 알아보자.

#### C++ static 멤버변수

```cpp
class User
{
    static int count;
};
```

             User 클래스
                 │
                 ▼
            static count
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      User a   User b   User c

객체마다 존재하는 것이 아니라 클래스에 하나만 존재함.
#### C++ static 멤버함수
```cpp
class Player 
{ 
	public: static void PrintCount(); 
};
```

객체 생성이 필요 없고 바로 호출이 가능하다.

```cpp
Player::PrintCount();
```

static 멤버함수는 특정 객체에 속한 함수가 아니기 때문이다. 
그러므로 static 멤버 함수에서는 일반 멤버 변수에 직접 접근할 수 없다.

```
class Player
{
    int hp;

public:
    static void foo()
    {
        hp = 100;  // ❌
    }
};

# 반면 static 변수에는 static 멤버함수에서 접근 가능하다
class Player
{
    static int count;

public:
    static void foo()
    {
        count++;   // ⭕
    }
};

```

우리가 대표적인 OOP 패턴인 싱글톤 패턴을 만들때도, 이러한 특징 덕분에 `static` 이 활용 된다. 프로세스 내에서 오직 "하나" 이기 때문이다.


```cpp
class Signleton
{
	public:
		static Singlenton & getInstance()
		{
			static Singleton sObj;
			return sObj;
		}
}
```

Scoped static을 활용하면 여러 쓰레드가 동시에 객체를 호출해도 mutex나 call_once 보호할 필요 없다. 

```cpp
#inlcude <iostream>
#include <thread>

class Cat
{
	public:
		Cat()
		{
			std::cout << "meow" << std::endl;
		}
};

void fn()
{
	static Cat cat;
}

int main()
{
	std::cout << "process start" << std::endl;
	std::thread t1(fn);
	std::thread t2(fn);
	
	t1.join();
	t2.join();
}
```
