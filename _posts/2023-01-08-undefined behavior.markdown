---
layout: post
title:  "Undefined behavior examples"
date:   2023-01-08 14:00:00 +0100
categories: jekyll update
---

- null-pointer dereference
{% highlight cpp %}
int* ptr = nullptr;
printf("%d", *ptr);
{% endhighlight %}


- signed integer overflow
{% highlight cpp %}
int var = std::numeric_limits<int>::max();
++var;
{% endhighlight %}


- calling a value-returning function which does not return
{% highlight cpp %}
int function()
{
    // some operations without return statement
}

int main()
{
    function();
}
{% endhighlight %}


- modifying a const value
{% highlight cpp %}
const int var = 1;
int* ptr = const_cast<int*>(&var);
*ptr = 2;
{% endhighlight %}


- overlapping objects with memcpy
{% highlight cpp %}
int arr[4]{};
memcpy(arr, arr, 4);
{% endhighlight %}


- memcpy when destination/source is a nullptr (even if count is 0)
{% highlight cpp %}
int arr[4]{};
int* ptr = nullptr;
memcpy(ptr, arr, 0);
{% endhighlight %}


- using pointers to objects whose lifetime is ended
{% highlight cpp %}
int* ptr = nullptr;
{
    int val = 5;
    ptr = &val;
}
printf("%d", *ptr);
{% endhighlight %}


- dereferencing a one-past-the-end element
{% highlight cpp %}
int arr[3]{ 1, 2, 3 };
int* ptr = arr + 3;
printf("%d", *ptr);
{% endhighlight %}


- accessing std::string when pos > size()
{% highlight cpp %}
std::string var{ "text" };
auto pos = var.size() + 1;
var[pos];
{% endhighlight %}


- modifying returned std::string reference when pos == size()
{% highlight cpp %}
std::string var{ "text" };
auto pos = var.size();
var[pos] = 'a';
{% endhighlight %}


- accessing std::string_view when pos >= size()
{% highlight cpp %}
std::string_view var{ "text" };
auto pos = var.size();
var[pos];
{% endhighlight %}


- using uninitialized local variable
{% highlight cpp %}
bool var;
if (var)
    printf("var is true");
{% endhighlight %}


- dereferencing a pointer which was passed to std::realloc
{% highlight cpp %}
int* var_1 = static_cast<int*>(std::malloc(sizeof(int)));
int* var_2 = static_cast<int*>(std::realloc(var_1, sizeof(int)));
*var_1 = 10;
{% endhighlight %}


- infinite loop without side-effects
{% highlight cpp %}
while (true) { }
{% endhighlight %}


- using a pointer after free
{% highlight cpp %}
int* ptr = new int{ 5 };
delete ptr;
printf("%d", *ptr);
{% endhighlight %}


- assumed expression is not evaluated to true
{% highlight cpp %}
void function(int var)
{
    [[assume(var > 0)]];
}

int main()
{
    function(-1);
}
{% endhighlight %}


- calling a function through a pointer to a different function type
{% highlight cpp %}
void function_1(int) { }
void function_2(short) { }

int main()
{
    void (*ptr_1)(int) = &function_1;
    void (*ptr_2)(short) = reinterpret_cast<decltype(&function_2)>(ptr_1);
    ptr_2(5);
}
{% endhighlight %}


- interpreting the bytes of an object as a value of a different type
{% highlight cpp %}
double var_1 = 1.0;
long long var_2 = *reinterpret_cast<long long*>(&var_1);
{% endhighlight %}


- subtracting pointers to different arrays
{% highlight cpp %}
int arr_1[10]{};
int arr_2[10]{};
std::ptrdiff_t diff = &arr_2[10] - &arr_1[0];
{% endhighlight %}


- in the bitwise shift operator expression when a value of the right operand is negative or is greater or equal to the number of bits in the promoted left operand
{% highlight cpp %}
char var_1 = 'A';
int var_2 = var_1 >> 8;
int var_3 = var_1 >> -1;
{% endhighlight %}


- calling a destructor twice
{% highlight cpp %}
class A{ };

int main()
{
    A var;
    var.~A(); // second call is when A leaves its current scope
}
{% endhighlight %}


- deleting an object through a pointer to a base with non-virtual destructor
{% highlight cpp %}
struct A
{
    ~A() { };
};

struct B : A{ };

int main()
{
    A* var = new B{};
    delete var;
}
{% endhighlight %}
