## inline 키워드
"함수 호출 오버헤드 없이 바로 실행해줘" 라고 컴파일러에게 말해주는 키워드 

"짧고 자주 호출되는 함수에 쓰는 것이 정석"

## explicit 키워드
```
class Foo {
public:
    Foo(int x) {}
};

Foo f = 10;   // 👈 int → Foo 암시적 변환 발생
```
explicit이 없으면 암시적 형변환이 발생한다. 
암시적 형변환을 막는 키워드  
`explicit` = **"이 생성자를 자동 형변환 용도로 쓰지 마라."**

## static 키워드

|구분|C|C++|
|---|:---|:---:|
|함수 내부 static|O|O|
|전역 static (파일 스코프)|O|O (비권장)|
|클래스 static 멤버|X|O|
|객체 공유 개념|X|O|
|namespace 대체|X|O|
