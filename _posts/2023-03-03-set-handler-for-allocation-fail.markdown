---
layout: post
title:  "{} initializer vs. () initializer"
date:   2023-03-05 23:00:00 +0100
categories: jekyll update
---

Using {}-initializer is related to the list initialization which doesn't allow narrowing conversions.

Type int is capable of holding type signed char, but type signed char cannot hold int:
```cpp
int b{ a };  // compiles with narrowing conversion
signed char a{ 'A' };
signed char c{ b };  // error
signed char d{ static_cast<signed char>(b) };  // compiles, explicit casting
```

Using {}-initializer detects assigning values which don't fit in a given type:
```cpp
signed char a = 256;  // compiles, but overflow and undefined behavior
signed char b{ 256 };  // error
```

Cannot assign signed variable type to an unsigned type:
```cpp
signed char a = -1;
unsigned short b = a;  // compiles, uses 2's complement method
unsigned short c{ a };  // error
```

Cannot assign double to float:
```cpp
float a = 1.0;
double b{ a };
float c{ b };  // error, narrowing conversion
```

Cannot assign floating-point to integer:
```cpp
float a = 1.0;
int b = a;  // compiles, narrowing conversion
int c{ a };  // error
```

Also, cannot assign integer to floating-point:
```cpp
int a = 1;
float b = a;
// float c{ a };  // error
```

Using {}-initializer detects narrowing conversions as below:
```cpp
static const double a{ 1.0 };  // global variable

struct A
{
    int b;

    A() : b(a) { }  // this constructor compiles with narrowing conversion
    // A() : b{ a } { }  // error
};
```

Structure `A` is an aggregate type:
```
{
	struct A
	{
		int x;
		int y;
	};

	A a{ 1 };  // aggregate initialization, x=1, y is default-initialized and its value is indeterminated, it's not necessarily 0
	A a2();  // function declaration
	A a3{ };  // zero-initialization, x=0, y=0
}
```

Distinguish initialization with () vs. {} when creating an object and such object has a constructor taking `std::initializer_list`:
```cpp
std::vector<int> a(5, 10);  // 5 elements, each has value 10
std::vector<int> b{ 5, 10 }; // 2 elements with value 5 and 10
```

Sometimes we can run into `the most vexing parse`:
```cpp
{
	std::string str();
	std::cout << str.size();  // error, str is not a std::string, it's a function declaration
	std::string str2{ };  // str2 is a std::string

	std::thread thr();
	thr.join();  // error, thr is not a std::thread, it's a function declaration
	std::thread thr2{ };  // thr2 is a std::thread
}
```

More:
- [https://en.cppreference.com/w/cpp/language/direct_initialization]

[https://en.cppreference.com/w/cpp/language/direct_initialization]: https://en.cppreference.com/w/cpp/language/direct_initialization
