---
title: "How to write a thread pool (Continue)"
date: 2020-06-17 21:30:00 -07:00
categories:
  - note
tags:
  - c++
  - thread
---

## Clarification  
Now we want input parameters in packaged_task  

## API definition  
Change the API definition
```
// Enqueue a task into thread pool and with a future object return
std::future<std::string> enqueue(std::packaged_task<std::string(int, int, int)> task, int id, int seq, int time);
```

## Implementation  
All we need is to change the type in the queue and handle the packaged_task and parameters properly
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

    std::future<std::string> enqueue(std::packaged_task<std::string(int, int, int)> task, int id, int seq, int time) {
        auto future = task.get_future();
        {
            std::unique_lock<std::mutex> lock(m);
            q.push(std::make_tuple(std::move(task), id, seq, time));
        }
        cv.notify_one();
        return future;
    }
private:
    std::mutex m;
    std::condition_variable cv;
    bool stopping;
    std::vector<std::thread> threads;
    std::queue<std::tuple<std::packaged_task<std::string(int, int, int)>, int, int, int>> q;

    void Start(int num_of_threads) {
        for (int i = 0; i < num_of_threads; i++) {
            threads.push_back(std::thread([=]() {
                while (true) {
                    std::packaged_task<std::string(int, int, int)> task;
                    int id;
                    int seq;
                    int time;
                    {
                        std::unique_lock<std::mutex> lock(m);
                        cv.wait(lock, [=]() {
                            return stopping || !q.empty();
                        });
                        if (stopping && q.empty()) {
                            break;
                        }
                        auto& [w, i, s, t] = q.front();
                        task = std::move(w);
                        id = i;
                        seq = s;
                        time = t;
                        q.pop();
                    }
                    if (task.valid()) {
                        task(id, seq, time);
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
