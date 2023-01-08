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


- calling a value-returning function without return statement
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


- using uninitialized local variable
{% highlight cpp %}
bool var;
if (var)
    printf("var is true");
{% endhighlight %}


- dereference a pointer which was passed to std::realloc
{% highlight cpp %}
int* var_1 = static_cast<int*>(std::malloc(sizeof(int)));
int* var_2 = static_cast<int*>(std::realloc(var_1, sizeof(int)));
*var_1 = 10;
{% endhighlight %}


- infinite loop without side-effects
{% highlight cpp %}
while (true) { }
{% endhighlight %}


- use a pointer after free
{% highlight cpp %}
int* ptr = new int{ 5 };
delete ptr;
printf("%d", *ptr);
{% endhighlight %}



