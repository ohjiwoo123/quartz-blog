병렬프로그래밍을 하게 되면, 고민해야하는 부분이 바로 
Race Condition, Data Race, Fasle Sharing이다. 
어떤 것들인지 알아보자.
### Race Condition (경쟁 상태)
Race Condition이란, 경쟁 때문에 결과가 실행 순서에 따라 달라지는 현상 이다.
**Race Condition = 넓은 개념의 "실행 타이밍/순서에 따른 경쟁 문제"**  

```cpp
#include <iostream>
#include <thread>

int count = 0;

void increment()
{
    for (int i = 0; i < 100000; i++)
        count++;
}

int main()
{
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << count << '\n';
}
```

결과를 increment 함수를 2개의 쓰레드로 실행하여, 200000을 예상할 수 있지만 결과 값은 제각각이다.
173421 185632 199874 ...
**실행 타이밍에 따라 결과가 달라지는 것**이 Race Condition(경쟁 상태)이다.

```
Thread 1        Thread 2
   │               │
   ├─ count 읽기 ──┤  → 둘 다 10을 읽음
   │               │
   ├─ 10 + 1       │
   │               ├─ 10 + 1
   │               │
   ├─ 11 저장      │
   │               ├─ 11 저장
   │               │
        결과 = 11
```

Race Condition을 피하는 방법 
1. Mutex 사용 (퍼포먼스에 좋진 않음)
2. Atomic 변수 사용 (퍼포먼스에 좋진 않음)
3. Share 되는 메모리 없이 구분하여 프로그래밍 (각각의 쓰레드가 자기만의 공간을 가지게끔)


### Data Race 
**Data Race = 메모리에 대한 동시 접근 때문에 발생하는 구체적인 경쟁 문제**
하나의 Shared 된 메모리에 여러 쓰레드가 동시에 접근하여 생긴 문제이다.
우리는 이런 것을 임계영역 이라고 하는데, 임계영역에 대한 보호조치가 안되면 아래 같은 코드에서는 count 변수가 예상된 결과 값이 나오지 않게 된다.

```cpp
#include <thread>

int data = 0;

void func()
{
    data++;   // 읽기 + 쓰기
}

int main()
{
    std::thread t1(func);
    std::thread t2(func);

    t1.join();
    t2.join();
}
```

data에 대한 변수에 대해서 동시접근 하게 됨.

```
Thread 1       Thread 2
   │              │
   ├─ data 읽기 ──┤
   │              ├─ data 읽기
   ├─ data 쓰기   │
   │              ├─ data 쓰기
```

그리고 C++에서는 이런 **Data Race가 발생하면 Undefined Behavior(정의되지 않은 동작) 이다.
### False Sharing (거짓참조)

**Cache Line(캐시라인)이란 ? 
- CPU 캐시 메모리에서 데이터가 전송되고 저장되는 가장 작은 기본 단위.
- 캐시라인의 사이즈는 x86 기준, 64바이트 이다.

0부터 1씩 1억번 더하는 프로그램이 있다.
주어진 전체 합을 쓰레드로 구한다고 가정하자.
```cpp
void fn1(std::vector <int> &nums, std::size_t beginIdx, std::size_t endIdx, int & sum)
{
	for(std::size_t idx=beginIdx; idx<endIdx; ++idx)
	{
		sum+= nums[idx];
	}
}

void fn2(std::vector <int> &nums, std::size_t beginIdx, std::size_t endIdx, int & sum)
{
	int localSum = 0;
	for(std::size_t idx=beginIdx; idx<endIdx; ++idx)
	{
		localSum += nums[idx];
	}
	sum = localSum;
}

int main()
{
	constexpr std::size_t count = 100000000;
	std::vector<int> nums(count,1);
	
	int sums[17];
	sums[0] = 0;
	sums[16] = 0;
	
	const auto start = std::chrono::strady_clock::now();
	
	std::thread t1(fn1,std::ref(nums),0,cout/2,std::ref(sums[0]));
	std::thread t2(fn1,std::ref(nums),cout/2,count,std::ref(sums[16]));
	
	t1.join();
	t2.join();
	
	const int sum = sums[0] + sums[16];
	const auto finish = std::chrono::strady_clock::now();
	std::chrono::duration<double> duration = finish -start;
	std::cout << "time in seconds:" << duration.count() << std::endl;
	std::cout << sum << std::endl;
}
```

False Sharing이 있는 경우,
2개의 쓰레드로 0부터 1씩 1억번 더하는 시간이 `0.75`초
(1개의 쓰레드로 더했을 때, 0.31초 보다 더 느리다..)
캐시 문제가 발생함...

False Sharing이 없는 경우,
2개의 쓰레드로 0부터 1씩 1억번 더하는 시간이 `0.166599`초 굉장히 빠른 결과를 확인할 수 있다.

![[Pasted image 20260902225853.png]]

```cpp
std::thread t1(fn2,std::ref(nums),0,cout/2,std::ref(sums[0]));
std::thread t2(fn2,std::ref(nums),cout/2,count,std::ref(sums[1]));
```
fn2를 사용해도 False Sharing 문제를 회피할 수 있다.
localSum 변수를 활용하여 회피됨.

![[Pasted image 20260902230357.png]]

