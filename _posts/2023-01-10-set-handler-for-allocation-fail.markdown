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
- or we can throw an exception which might be caught by the caller
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
