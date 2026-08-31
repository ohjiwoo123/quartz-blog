
```
#include <cstring>

void storeByValue(std::string s)
{
	std::string b = s;
};

void storeByLRef(std::string &s)
{
	std::string b = s;
};

void storeByRRef(std::string && s)
{
	std::string b = s;  // wrong 
    // s를 계속 접근할 수 있기 때문에 틀린 문법이 됨.  
    // ex) std::cout << s << std::endl;
    // 인자의 s는 r-value 이기 때문에, 계속 접근하면 안 된다.
    
    std::string b = std::move(s);  // correct 
};

int main()
{
	std::string a = "abc";
	storeByValue(a);
	storeByLRef(a);
	storeByRRef("abc);
}
```


### storeByValue의 경우  

|storeByValue|
|---|
|a|
|s|
|b|
|-|
|"abc"|
|"abc"|
|"abc"|

결론 : 2 copy가 일어남.    

### storeByLRef의 경우  

|storeByLRef|
|---|
|a|
|s|
|b|
|-|
|"abc"|
|"abc"|

a-> "abc" s -> "abc" (같은 abc를 가르킴)  
b->"abc"  

결론 : 1 copy가 일어남.   

### storeByRRef의 경우
|storeByRRef|
|---|
|&&s|
|b|
|-|
|"abc"|
&&s -> "abc"를 가르키다가 b가 -> "abc"를 가르킴  

결론 : 0 copy가 일어남.


최종적으로 지역함수 안에서 지역변수로 인자(arguments)로 넘겨주고 싶을 때의 경우만 산정한다면, 
r-value를 넘겨줌으로서 0 copy를 통해 프로그램 최적화가 가능하다.   


추가로 이러한 구조도 가능하다.   

```
#include <string>
#include <iostream>

// 우리의 목적은 L-VALUE를 넘겨줄 땐 한번의 COPY만 일어나고,
// R-VALUE를 넘겨줄 땐 0 COPY가 일어나는 것.  

class Cat
{
    public:
        /* L-VALUE, R-VALUE 모두 1 COPY가 일어남.  */
        void setName(const std::string &name)
        {
            mName = std::move(name);
        }
        /* L-VALUE 1 COPY, R-VALUE 0 COPY가 일어남. */
        void setName(std::string name)
        {
            mName = std::move(name);
        }
    private:
    std::string mName;
};

int main()
{
    Cat kitty;
    std::string s = "kitty";

    kitty.setName(s);
    std::cout << s << std::endl;
    if (s.empty()==true)
    {
        std::cout << "s is empty" << std::endl;
    }
    // kitty.setName("nabi"); // 0 copy 
}

```
