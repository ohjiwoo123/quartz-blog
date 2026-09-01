객체는 메모리에 어떤식으로 할당이 되는걸까? 

아래와 같은 객체가 있다고 가정하자.

```cpp
#include <iostream>

class Cat
{
public:
	void speak();
private:
	double d8; // 8 bytes
	int i4a; // 4 bytes
	int i4b; // 4 bytes
}

# sizeof(Cat) = 16 bytes

class Cat
{
public:
	void speak();
private:
	int i4a; // 4 bytes
	double d8; // 8 bytes
	int i4b; // 4 bytes
}

# sizeof(Cat) = 24 bytes
```
위의 Cat 객체는 16바이트, 아래의 Cat 객체는 24바이트가 나온다. 
이유가 무엇일까? 이유는 구조체와 마찬가지로 메모리 Alignment Rule이 있기 때문이다.
Member Variable은 그 size의 배수에 맞게 시작되어야 한다.

자세한 메모리에 할당된 그림은 아래와 같다
![[Pasted image 20260901232059.png]]

Cat 객체가 만약에 24 바이트라고 가정하고, 배열로 만든다고 가정했을 때,
Cache Line이라는 하드웨어 구조 때문에
잘라진 조각이 각각 다른 코어에 들어갈 수 있다.
![[Pasted image 20260901232122.png]]
그렇기 때문에, 이런 부분을 고려해서 객체를 정렬해야 할 수도 있을 것이다.

```cpp
#include <iostream>

class alignas(32) Cat
{
public:
	void speak();
private:
	char c1; // 8 bytes
	int i4a; // 4 bytes
	int i4b; // 4 bytes
	double d8; // 8 bytes
}

int main()
{
	Cat stackCat;
	Cat cats[100];
}
```

클래스에 패딩을 없앨 수는 있지만, 퍼포먼스에 문제가 생길 수 있기 때문에 사용하지 않는 것이 좋다.

FALSE SHARING과도 관련이 있는 중요한 내용이다.
우린 구조체에도 PADDING이 있다는 것을 잘 알고 있다 .
[[구조체 패딩]]  

똑같이 CLASS에서도 PADDING이 존재한다는 것을 배웠습니다. 
그리고 객체를 설계할 때, 예를 들어 여러 쓰레드에서 사용하게 된다면, 캐시 라인에 맞게 정렬해야 FALSE SHARING(거짓참조) 문제를 안 겪을 수 있을 것이다.

