---
layout: post
title:  "Set handler for allocation fail"
date:   2023-01-09 14:00:00 +0100
categories: jekyll update
---

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
