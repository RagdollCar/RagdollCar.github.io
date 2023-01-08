---
layout: post
title:  "Undefined behavior examples"
date:   2023-01-08 14:00:00 +0100
categories: jekyll update
---
Undefined behavior

1. null-pointer dereference
{% highlight cpp %}
int* ptr = nullptr;
printf("%d", *ptr);
{% endhighlight %}

2. signed integer overflow
{% highlight cpp %}
int var = std::numeric_limits<int>::max();
++var;
{% endhighlight %}

3. calling a value-returning function without return statement
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

4. modifying a const value
{% highlight cpp %}
const int var = 1;
int* ptr = const_cast<int*>(&var);
*ptr = 2;
{% endhighlight %}

5. overlapping objects with memcpy
{% highlight cpp %}
int arr[4]{};
memcpy(arr, arr, 4);
{% endhighlight %}

6. memcpy when destination/source is a nullptr (even if count is 0)
{% highlight cpp %}
int arr[4]{};
int* ptr = nullptr;
memcpy(ptr, arr, 0);
{% endhighlight %}
