---
layout: post
title:  "Memory allocation failed"
date:   2023-01-10 23:00:00 +0100
categories: jekyll update
---

Sometimes memory allocation fails. It is usually caused by the missing memory. In such situations `operator new` throws an exception. But before it happens, a special function is called. We have an ability to control this behavior by using a function `std::set_new_handler`. So, let's start.

First create a static variable for a reserved buffer. It will be freed when we're out of memory.
```cpp
static uint8_t* reservedBuffer;
```

Now we must define a function which will be called when the memory allocation fails. Such function may check if there is some memory that can be freed. If so, then deallocate it. Then we can:
- just return without throwing an exception, it will cause invoking a new try of allocating memory which previously failed
- or throw an exception which might be caught by the caller.
We can also set a new handler or unset it by passing `nullptr` to the function `std::set_new_handler`.

Assuming that we have a global buffer with some reserved space, firstly we deallocate such buffer and just return from the function. For the second time of calling `my_handler` the global buffer will be `nullptr`, no memory will be available so we end up with throwing an exception.

```cpp
void my_handler()
{
    printf("Failed to allocate memory\n");

    if (reservedBuffer) {
        delete[] reservedBuffer;
        reservedBuffer = nullptr;
        return;
    }

    throw std::bad_alloc{};
}
```

To set such function to be called for each memory allocation fail, use the function below:
```cpp
std::set_new_handler(my_handler);
```

What is left is to create a simple while loop allocating memory. When such allocation fails then our function `my_handler` is called. For the first time we deallocate memory from the global buffer and let the loop continue. For the second time we throw an exception so the caller can catch it and do some other stuff.

The whole program is as follows:
```cpp
#include <cstdio>
#include <new>
#include <vector>

static constexpr std::size_t KB{ 1024 };
static constexpr std::size_t MB{ 1024 * KB };
static constexpr std::size_t GB{ 1024 * MB };

static uint8_t* reservedBuffer;
std::vector<uint8_t*> vec;

void my_handler()
{
    printf("Failed to allocate memory\n");

    if (reservedBuffer) {
        delete[] reservedBuffer;
        reservedBuffer = nullptr;
        return;
    }

    throw std::bad_alloc{ };
}

int main()
{
    reservedBuffer = new uint8_t[1 * GB]{ };
    std::set_new_handler(my_handler);

    try {
        while (true) {
            uint8_t* ptr = new uint8_t[MB]{ };
            vec.push_back(ptr);
        }
    }
    catch (const std::bad_alloc& ex) {
        printf("%s\n", ex.what());
    }

    for (auto elem : vec)
        delete[] elem;

    if (reservedBuffer)
        delete[] reservedBuffer;
}
```

Refs:
- [https://en.cppreference.com/w/cpp/memory/new/set_new_handler]

[https://en.cppreference.com/w/cpp/memory/new/set_new_handler]: https://en.cppreference.com/w/cpp/memory/new/set_new_handler
