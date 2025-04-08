---
title: Thread safe stack
date: 2025-04-08 10:00:00 +0800
categories: [Programming]
tags: [C++, Multithread]
description: Implement thread safe stack
---


# Thread safe stack
Almost the same as thread safe queue.[Check the thread safe queue]({{ '/posts/2025-04-08-concurrency_1/' | relative_url }})

```cpp
#include <iostream>
#include <condition_variable>
#include <mutex>
#include <stack>
#include <optional>
#include <thread>
#include <future>
#include <chrono>

template<typename T>
class ThreadSafeStack {
    std::mutex m_;
    std::condition_variable con_;
    std::stack<T> stack_;

public:
    ThreadSafeStack() = default;

    //std::mutex is non-copyable
    ThreadSafeStack(const ThreadSafeStack& other) = delete;
    ThreadSafeStack& operator=(const ThreadSafeStack& other) = delete;

    ThreadSafeStack(ThreadSafeStack&& other) noexcept {
            std::lock_guard<std::mutex> l(other.m_);
            stack_ = std::move(other.stack_);
    }

    ThreadSafeStack& operator=(ThreadSafeStack&& other) noexcept{
        if(this != &other) {
            std::scoped_lock<std::mutex> l(m_, other.m_);
            stack_ = std::move(other.stack_);
        }

        return *this;
    }

    template<typename U>
    void push(U&& item) {
        {
            std::lock_guard<std::mutex> l(m_);
            stack_.push(std::forward<U>(item));
        }
        con_.notify_one();
    }

    //blocking pop, using con_ and lock
    T pop() {
        std::unique_lock<std::mutex> l(m_);
        con_.wait(l, [this]() { return !stack_.empty();});
        T item = std::move(stack_.top()); //safe since pop later, also efficient if T is large
        stack_.pop();
        return item;
    }

    //non blocking pop
    std::optional<T> try_pop() {
        std::lock_guard<std::mutex> l(m_);
        if(stack_.empty()) return std::nullopt;
        T item = std::move(stack_.top());
        stack_.pop();
        return item;
    }

    //blocking timeout
    template<typename Rep, typename Period>
    std::optional<T> pop_for(std::chrono::duration<Rep, Period> timeout){
        std::unique_lock<std::mutex> l(m_);
        if(!con_.wait_for(l, timeout, [this](){return !stack_.empty();}) )
            return std::nullopt;
        T item = std::move(stack_.top());
        stack_.pop();
        return item;
    }

    bool empty() const {
        std::lock_guard<std::mutex> l(m_);
        return stack_.empty();
    }

    size_t size() const {
        std::lock_guard<std::mutex> l(m_);
        return stack_.size();
    }
};
template<typename T>
void pushq(ThreadSafeStack<T>& q, int a) {
    std::this_thread::sleep_for(std::chrono::seconds(3));
    q.push(a);
}

template<typename T>
T popq(ThreadSafeStack<T>& q) {
    return q.pop();
}

template<typename T>
std::optional<T> try_popq(ThreadSafeStack<T>& q) {
    return q.try_pop();
}


int main() {
    ThreadSafeStack<int> tsq;

    std::thread t1(pushq<int>, std::ref(tsq), 3);
    std::future<int> f1 = std::async(std::launch::async, popq<int>, std::ref(tsq));
    std::future<std::optional<int>> f2 = std::async(std::launch::async, try_popq<int>, std::ref(tsq));
    std::cout<<f1.get()<<std::endl;

    auto result = f2.get();
    if(result) std::cout<<*result<<std::endl;
    else std::cout<<"the stack is empty!"<<std::endl;
    t1.join();
    return 0;
}
```





