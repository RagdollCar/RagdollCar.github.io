---
layout: post
title:  "Set handler for allocation fail"
date:   2023-01-10 23:00:00 +0100
categories: jekyll update
---

First, define some static variables representing units of memory size.
```cpp
static constexpr std::size_t KB{ 1024 };
static constexpr std::size_t MB{ 1024 * KB };
static constexpr std::size_t GB{ 1024 * MB };
```

Then create a static variable for a reserved buffer. It will be freed when we're out of memory.
```cpp
static uint8_t* reservedBuffer;
```

Now we must define a function which will be called when the memory allocation fails. Such function may check if there is some memory that can be freed. If so, then deallocate it. Now we have two choices:
- we can just return without throwing an exception, it will cause invoking a new try of allocating memory which previously failed
- or we can throw an exception which might be caught by the caller.
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

#define MB 1024 * 1024

static char* reservedBuffer;
std::vector<char*> vec;

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

int main()
{
    reservedBuffer = new char[100 * MB];
    std::set_new_handler(my_handler);

    try {
        while (true)
        {
            char* ptr = new char[100 * MB];
            vec.push_back(ptr);
        }
    }
    catch (const std::bad_alloc& ex) {
        printf("%s", ex.what());
    }

    for (auto elem : vec)
        delete[] elem;

    if (reservedBuffer)
        delete[] reservedBuffer;
}
```

Refs:
[0] https://en.cppreference.com/w/cpp/memory/new/set_new_handler