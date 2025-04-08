---
title: Thread safe queue
date: 2025-04-08 10:00:00 +0800
categories: [Programming]
tags: [C++, Multithread]
description: Implement thread safe queue
---


# Thread safe queue
To implement a thread safe queue, the followings are considered:
1. Since mutex locks the shared resource, it is not copyable. The copy constructor and copy assignment operator should be deleted.
2. For blocking pop(always return valid element of the queue), using unique_block and condition variable to pop out a valid element, which will wait unitil the queue has at least one element.
3. For non-blocking pop, it will lock the resource and check if the queue is empty or not, so it wll return std::optional value,depending on the queue status( return nullopt if empty, or valid element ).
4. For timed pop, comparing to blocking pop, it will only wait for specific timeout, and then return either nullopt or valid element.


```cpp



#include <iostream>
#include <condition_variable>
#include <mutex>
#include <queue>
#include <optional>
#include <thread>
#include <future>
#include <chrono>

template<typename T>
class ThreadSafeQueue {
    std::mutex m_;
    std::condition_variable con_;
    std::queue<T> queue_;

public:
    ThreadSafeQueue() = default;

    //std::mutex is non-copyable
    ThreadSafeQueue(const ThreadSafeQueue& other) = delete;
    ThreadSafeQueue& operator=(const ThreadSafeQueue& other) = delete;

    ThreadSafeQueue(ThreadSafeQueue&& other){
            std::lock_guard<std::mutex> l(other.m_);
            queue_ = std::move(other.queue_);
    }

    ThreadSafeQueue& operator=(ThreadSafeQueue&& other) {
        if(this != &other) {
            std::lock_guard<std::mutex> l1(m_);
            std::lock_guard<std::mutex> l2(other.m_);
            queue_ = std::move(other.queue_);
        }

        return *this;
    }

    template<typename U>
    void push(U&& item) {
        {
            std::lock_guard<std::mutex> l(m_);
            queue_.push(std::forward<U>(item));
        }
        con_.notify_one();
    }

    //blocking pop, using con_ and lock
    T pop() {
        std::unique_lock<std::mutex> l(m_);
        con_.wait(l, [this]() { return !queue_.empty();});
        T item = std::move(queue_.front()); //safe since pop later, also efficient if T is large
        queue_.pop();
        return item;
    }

    //non blocking pop
    std::optional<T> try_pop() {
        std::lock_guard<std::mutex> l(m_);
        if(queue_.empty()) return std::nullopt;
        T item = std::move(queue_.front());
        queue_.pop();
        return item;
    }

    //blocking timeout
    template<typename Rep, typename Period>
    std::optional<T> pop_for(std::chrono::duration<Rep, Period> timeout){
        std::unique_lock<std::mutex> l(m_);
        if(!con_.wait_for(l, timeout, [this](){return !queue_.empty();}) )
            return std::nullopt;
        T item = std::move(queue_.front());
        queue_.pop();
        return item;
    }

    bool empty() const {
        std::lock_guard<std::mutex> l(m_);
        return queue_.empty();
    }

    size_t size() const {
        std::lock_guard<std::mutex> l(m_);
        return queue_.size();
    }
};
template<typename T>
void pushq(ThreadSafeQueue<T>& q, int a) {
    std::this_thread::sleep_for(std::chrono::seconds(3));
    q.push(a);
}

template<typename T>
T popq(ThreadSafeQueue<T>& q) {
    return q.pop();
}

template<typename T>
std::optional<T> try_popq(ThreadSafeQueue<T>& q) {
    return q.try_pop();
}


int main() {
    ThreadSafeQueue<int> tsq;

    std::thread t1(pushq<int>, std::ref(tsq), 3);
    std::future<int> f1 = std::async(std::launch::async, popq<int>, std::ref(tsq));
    std::future<std::optional<int>> f2 = std::async(std::launch::async, try_popq<int>, std::ref(tsq));
    std::cout<<f1.get()<<std::endl;

    auto result = f2.get();
    if(result) std::cout<<*result<<std::endl;
    else std::cout<<"the queue is empty!"<<std::endl;
    t1.join();
    return 0;
}
```



