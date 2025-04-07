---
title: Async, future and packaged_task in concurrency
date: 2025-04-06 10:00:00 +0800
categories: [Programming]
tags: [C++, Multithread]
description: Basic usage of the async, future and packaged_task
---


# Async, future and packaged_task
Async will launch the job/task for run in async style once the keyword std::launch::async is specified. Otherwise, both sync and async can be possible(unpredictable). The return value will be wrapped within a future object, and the value can be extracted with .get() function. Packaged\_task wrap a function and leave it for later run with thread. Also, the return value can be extracted through the .get\_future() method. 
```cpp
#include <iostream>
#include <future>
#include <mutex>
#include <vector>

std::mutex mu;
int shared_num;

int increment(int i) {
    std::lock_guard<std::mutex> l(mu);

    shared_num += i;

    return i * i;
}

int get_back(int i) {
    return i;
}

int main() {
    std::vector<std::future<int>> futures;

    for(int i = 0; i < 5; ++i) {
        futures.push_back( std::async(std::launch::async, increment, i) );
    }

    for(auto& f:futures) {
        std::cout<<f.get()<<std::endl;
    }

    std::cout<<shared_num<<std::endl;

    std::packaged_task<int(int)> p(get_back);

    std::future<int> f = p.get_future();

    std::thread t1(std::move(p), 3);

    auto result = f.get();

    std::cout<<result<<std::endl;

    t1.join();



    return 0;
}
```

