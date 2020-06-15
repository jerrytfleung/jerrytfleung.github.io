---
title: "Copy constructor of thread is deleted by design"
date: 2020-06-14 20:30:00 -07:00
categories:
  - note
tags:
  - c++
  - thread
---

If we want to create a vector of threads, we cannot do this.

```
auto thread_function = []() {
  std::cout << "This is thread function" << std::endl;
};

int n = 100;
std::vector<std::thread> threads(n, std::thread(thread_function));
```

This leads to compilation errors.
```
error: calling a private constructor of class 'std::__1::thread'
            ::new((void*)__p) _Up(_VSTD::forward<_Args>(__args)...);
```

It is because the copy constructor of thread is deleted by design. i.e. thread is movable only but not copyable.

```
thread() noexcept;(since C++11)
thread( thread&& other ) noexcept; (since C++11)
template< class Function, class... Args >
explicit thread( Function&& f, Args&&... args ); (since C++11)
thread(const thread&) = delete; (since C++11)
```

The correct way is
```
auto thread_function = []() {
  std::cout << "This is thread function" << std::endl;
};

std::vector<std::thread> threads;
for (int i = 0; i < 100; i++) {
  threads.push_back(thread(thread_function));
}
```
```push_back``` function of vector has both copyable and movable version, that's why it compiles.
```
void push_back( const T& value );
void push_back( T&& value ); (since C++11)
```
Still you can use ```std::move``` to produce the rvalue reference of ```thread(thread_function)```.

How about copy assignment operator of thread?  
There is no copy assignment operator but move assignment operator in thread class.
```
thread& operator=( thread&& other ) noexcept; (since C++11)
```
