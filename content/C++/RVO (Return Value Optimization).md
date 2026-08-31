```
#include <string>

std::string getString()
{
	s = "nocodeprogram";
    return s; // RVO, 0 copy 
}


std::string getString(std::string a, bool b)
{
	if(b)
    {
    	a = "nocodeprogram";
    }
    return a; // No RVO, 0 copy
}

int main()
{
	std::string a = getString();
    return 0;
}
```

더 자세히 알고 싶다면  
https://en.cppreference.com/w/cpp/language/copy_elision  