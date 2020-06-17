---
title: "How to write a thread pool"
date: 2020-06-16 23:20:00 -07:00
categories:
  - note
tags:
  - c++
  - thread
---

1. Clarification  
a. We can specify number of threads in the pool  
b. We can enqueue tasks (For simplicity, task takes no argument and returns an integer) into the thread pool  
c. We can retrieve the result of task from the thread pool  

2. API definition  
```
// Start the thread pool
void start(int num_of_threads);
// Stop the thread pool
void stop();
// Enqueue a task into thread pool and with a future object return
std::future<int> enqueue(packaged_task<int()> task);
```

3. Implementation  
We need the following to create the most basic form of a thread pool
a. a mutex (m)
b. a condition_variable (cv)
c. a bool (stopping)
d. a vector of thread (threads)
```
class ThreadPool {
public:
    explicit ThreadPool(int num_of_threads) {
        stopping = false;
        Start(num_of_threads);
    }

    virtual ~ThreadPool() {
        Stop();
    }
private:
    std::mutex m;
    std::condition_variable cv;
    bool stopping;
    std::vector<std::thread> threads;
    std::queue<std::packaged_task<void()>> q;

    void Start(int num_of_threads) {
        for (int i = 0; i < num_of_threads; i++) {
            threads.push_back(std::thread([=]() {
                while (true) {
                    std::unique_lock<std::mutex> lock(m);
                    /**
                     * The wait of condition_variable will block the current thread, will unlock the lock, waiting for the stopping to be true.
                     * If other thread notify_one() or notify_all() and stopping is false, it will wait again.
                     */
                    cv.wait(lock, [=]() {
                        return stopping;
                    });
                    if (stopping) {
                        break;
                    }
                }
            }));
        }
    }

    void Stop() {
        {
            std::unique_lock<std::mutex> lock(m);
            stopping = true;
        }
        cv.notify_all();
        for (auto &th : threads) {
            th.join();
        }
    }
};
```
We need the following in order to support adding tasks
a. a queue
```
class ThreadPool {
public:
    explicit ThreadPool(int num_of_threads) {
        stopping = false;
        Start(num_of_threads);
    }

    virtual ~ThreadPool() {
        Stop();
    }

    void enqueue(std::packaged_task<void()> task) {
        {
            std::unique_lock<std::mutex> lock(m);
            q.push(std::move(task));
        }
        cv.notify_one();
    }
private:
    std::mutex m;
    std::condition_variable cv;
    bool stopping;
    std::vector<std::thread> threads;
    std::queue<std::packaged_task<void()>> q;

    void Start(int num_of_threads) {
        for (int i = 0; i < num_of_threads; i++) {
            threads.push_back(std::thread([=]() {
                while (true) {
                    std::packaged_task<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(m);
                        /**
                         * The wait of condition_variable will block the current thread, will unlock the lock, waiting for the stopping to be true.
                         * If other thread notify_one() or notify_all() and stopping is false, it will wait again.
                         */
                        cv.wait(lock, [=]() {
                            return stopping || !q.empty();
                        });
                        if (stopping && q.empty()) {
                            break;
                        }
                        task = std::move(q.front());
                        q.pop();
                    }
                    if (task.valid()) {
                        task();
                    }
                }
            }));
        }
    }

    void Stop() {
        {
            std::unique_lock<std::mutex> lock(m);
            stopping = true;
        }
        cv.notify_all();
        for (auto &th : threads) {
            th.join();
        }
    }
};
```
We can finalise the implementation by adding a std::future in the return of enqueue function and change ```void``` to ```int```
```
class ThreadPool {
public:
    explicit ThreadPool(int num_of_threads) {
        stopping = false;
        Start(num_of_threads);
    }

    virtual ~ThreadPool() {
        Stop();
    }

    std::future<int> enqueue(std::packaged_task<int()> task) {
        auto future = task.get_future();
        {
            std::unique_lock<std::mutex> lock(m);
            q.push(std::move(task));
        }
        cv.notify_one();
        return future;
    }
private:
    std::mutex m;
    std::condition_variable cv;
    bool stopping;
    std::vector<std::thread> threads;
    std::queue<std::packaged_task<int()>> q;

    void Start(int num_of_threads) {
        for (int i = 0; i < num_of_threads; i++) {
            threads.push_back(std::thread([=]() {
                while (true) {
                    std::packaged_task<int()> task;
                    {
                        std::unique_lock<std::mutex> lock(m);
                        /**
                         * The wait of condition_variable will block the current thread, will unlock the lock, waiting for the stopping to be true.
                         * If other thread notify_one() or notify_all() and stopping is false, it will wait again.
                         */
                        cv.wait(lock, [=]() {
                            return stopping || !q.empty();
                        });
                        if (stopping && q.empty()) {
                            break;
                        }
                        task = std::move(q.front());
                        q.pop();
                    }
                    if (task.valid()) {
                        task();
                    }
                }
            }));
        }
    }

    void Stop() {
        {
            std::unique_lock<std::mutex> lock(m);
            stopping = true;
        }
        cv.notify_all();
        for (auto &th : threads) {
            th.join();
        }
    }
};
```
