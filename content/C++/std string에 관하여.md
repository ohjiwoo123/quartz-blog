## Char (C) vs std::string (C++)
cpp의 std::string에 대해서 깊게 공부해보자.   

단순하게 우리는 C에서 char를 사용하다가 C++에서 std::string으로 대체하면서 편하다고만 생각하지만,  
정확하게 어떻게 동작하는지에 대해서는 아는 사람이 많이 없다.   

우선 std::string은 char와 비교하면 무거운 동작? 이라고 할 수 있겠다.   
기본적으로 스택을 사용하는 char 변수와 달리, std::string은 `heap`을 사용한다.  

stack에 저장된 객체는 크기를 변경할 수 없다.   
std::string은 push_back 같은 동적으로 변경되는 작업이 수반되고, 그에 따라서  
heap에 할당하여 사용하는 것이다.   

결국엔 std::string 변수를 선언하게 되면,  
힙에서 할당을하고 스택에 저장을 하는 식이 되는 것이다.  

## SSO Small String Optimization
std::string에서는 작은 사이즈의 스트링에 대해서 최적화작업을 진행한다. 주로 23bytes + 1 (null-terminator)보다 작으면 힙에 할당되지 않고 스택에 할당하여 처리한다.  
이보다 큰 경우에는 힙에 할당된다.  


## std::string의 크기는?  

```
std::string str = "nocope";
sizeof(str)  = 32
str.size() = 6
```
컴파일러마다 크기가 다른데, 32바이트 또는 40바이트라고 한다.  


```cpp
struct string {
    char*      pointer;      // 8
    size_t     length;       // 8

    union {
        char local_buffer[16];
        size_t allocated_capacity;
    };
}; // 총 32 bytes
```

[[string 메모리 구조]]

참고자료 :  
https://stackoverflow.com/questions/42049778/where-is-a-stdstring-allocated-in-memory
https://en.cppreference.com/w/cpp/string/basic_string