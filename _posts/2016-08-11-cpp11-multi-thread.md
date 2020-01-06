---
layout: cnpost
title: "翻译：C++ 11 线程、锁和条件变量"
date: 2016-08-11 01:00:00
categories: cn
tags: 翻译 C/C++
---

原文：[
C++11 threads, locks and condition variables](http://www.codeproject.com/Articles/598695/Cplusplus-threads-locks-and-condition-variables)

* content
{:toc}

### 线程

`std::thread` 类， 位于 `<thread>` 头文件，实现了线程操作。`std::thread` 可以和普通函数和 lambda 表达式搭配使用。它还允许向线程的执行函数传递任意多参数。

```cpp
#include <thread>
 
 void func()
{
   // do some work
}
 
int main()
{
   std::thread t(func);
   t.join();
   return 0;
}
```

上面的例子中，`t` 是一个线程实例，函数 `func()` 在该线程运行。调用 `join()` 函数是为了阻塞当前线程（此处即主线程），直到 `t` 线程执行完毕。线程函数的返回值都会被忽略，但线程函数接受任意数目的输入参数。

```cpp
void func(int i, double d, const std::string& s)
{
    std::cout << i << ", " << d << ", " << s << std::endl;
}

int main()
{
   std::thread t(func, 1, 12.50, "sample");
   t.join();
       
   return 0;

}
}
```

虽然可以向线程函数传递任意多参数，但都必须以值传递。如果需以引用传递，则必须以 `std::ref` 或 `std::cref` 封装，如下例所示：


```cpp
void func(int& a)
{
   a++;

}
 
 int main()
{
   int a = 42;
   std::thread t(func, std::ref(a));
   t.join();

   std::stringcout << a << std::endl;

   return 0;
}
```

这个程序会打印 `43`，但如果不用 `std::ref` 封装，则输出会是 `42`。

除了 `join` 函数，这个类还提供更多的操作：

- `swap`：交换两个线程实例的句柄

- `detach`：允许一个线程继续独立于线程实例运行；detach 过的线程不可以再 join

```cpp
int main()
{
    std::thread t(funct);
    t.detach();

    return 0;
}
```

一个重要的知识点是，如果一个线程函数抛出异常，并不会被常规的 `try-catch` 方法捕获。也就是说，下面的写法是不会奏效的：

```cpp
try
{
    std::thread t1(func);
    std::thread t2(func);

    t1.join();
    t2.join();
}
catch(const std::exception& ex)
{
    std::cout << ex.what() << std::endl;
}
```

要追踪线程间的异常，你可以在线程函数内捕获，暂时存储在一个稍后可以访问的结构内。

```cpp
std::mutex                       g_mutex;
std::vector<std::exception_ptr>  g_exceptions;

void throw_function()
{
   throw std::exception("something wrong happened");
}

void func()
{
   try
   {
      throw_function();
   }
   catch(...)
   {
      std::lock_guard<std::mutex> lock(g_mutex);
      g_exceptions.push_back(std::current_exception());
   }
}

int main()
{
   g_exceptions.clear();

   std::thread t(func);
   t.join();

   for(auto& e : g_exceptions)
   {
      try 
      {
         if(e != nullptr)
         {
            std::rethrow_exception(e);
         }
      }
      catch(const std::exception& e)
      {
         std::cout << e.what() << std::endl;
      }
   }

   return 0;
}
```

关于捕获和处理异常，更深入的信息可以参看 [Handling C++ exceptions thrown from worker thread in the main thread](http://binglongx.wordpress.com/2010/01/03/handling-c-exceptions-thrown-from-worker-thread-in-the-main-thread/) 和 [How can I propagate exceptions between threads?](http://stackoverflow.com/questions/233127/how-can-i-propagate-exceptions-between-threads) 。

此外，值得注意的是，<thread> 头文件还在 `std::this_thread` 命名空间下提供了一些辅助函数：

- [get_id](http://en.cppreference.com/w/cpp/thread/get_id): 返回当前线程的 id

- [yield](http://en.cppreference.com/w/cpp/thread/yield): 告知调度器运行其他线程，可用于当前处于繁忙的等待状态

- [sleep_for](http://en.cppreference.com/w/cpp/thread/sleep_for)：给定时长，阻塞当前线程

- [sleep_until](http://en.cppreference.com/w/cpp/thread/sleep_until)：阻塞当前线程至给定时间点


### 锁

在上个例子中，我们需要对 `g_exceptions` 这个 vector 的访问进行同步处理，确保同一时刻只有一个线程能向它插入新的元素。为此我使用了一个 mutex 和一个锁（lock）。mutex 是同步操作的主体，在 C++ 11 的 `<mutex>` 头文件中，有四种风格的实现：

- [mutex](http://en.cppreference.com/w/cpp/thread/mutex)：提供了核心的 `lock()` `unlock()` 方法，以及当 mutex 不可用时就会返回的非阻塞方法 `try_lock()`

- [recursive_mutex](http://en.cppreference.com/w/cpp/thread/recursive_mutex)：允许同一线程内对同一 mutex 的多重持有

- [timed_mutex](http://en.cppreference.com/w/cpp/thread/timed_mutex)： 与 `mutex` 类似，但多了 `try_lock_for()` `try_lock_until()` 两个方法，用于在特定时长里持有 mutex，或持有 mutex 直到某个特定时间点

- [recursive_timed_mutex](http://en.cppreference.com/w/cpp/thread/recursive_timed_mutex)：`recursive_mutex` 和 `timed_mutex` 的结合

下面是一个使用 `std::mutex` 的例子（注意 `get_id()` 和 `sleep_for()` 两个辅助方法的使用，上文已有提及）。

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>
 
std::mutex g_lock;
 
void func()
{
    g_lock.lock();
 
    std::cout << "entered thread " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(rand() % 10));
    std::cout << "leaving thread " << std::this_thread::get_id() << std::endl;
 
    g_lock.unlock();
}
 
int main()
{
    srand((unsigned int)time(0));
 
    std::thread t1(func);
    std::thread t2(func);
    std::thread t3(func);
 
    t1.join();
    t2.join();
    t3.join();
 
    return 0;
}
```

输出如下：

```
entered thread 10144
leaving thread 10144
entered thread 4188
leaving thread 4188
entered thread 3424
leaving thread 3424
```

`lock()` `unlock()` 两个方法应该很好懂，前者锁住 mutex，如果该 mutex 不可用，则阻塞线程；稍后，后者解锁线程。

下面一个例子展示了一个简单的线程安全的容器（内部使用了 `std::vector`）。该容器提供用于添加单一元素的 `add()`方法，以及添加多个元素的 `addrange()` 方法（内部调用 `add()` 实现）。

**注意**：尽管如此，下面会指出，由于 `va_args` 的使用等原因，这个容器并非真正线程安全。此外，`dump()` 方法不应属于容器，在实际实现中它应该作为一个独立的辅助函数。这个例子的目的仅仅是展示 mutex 的相关概念，而非实现一个完整的线程安全的容器。


```cpp
template <typename T>
class container 
{
    std::mutex _lock;
    std::vector<T> _elements;
public:
    void add(T element) 
    {
        _lock.lock();
        _elements.push_back(element);
        _lock.unlock();
    }
 
    void addrange(int num, ...)
    {
        va_list arguments;
 
        va_start(arguments, num);
 
        for (int i = 0; i < num; i++)
        {
            _lock.lock();
            add(va_arg(arguments, T));
            _lock.unlock();
        }
 
        va_end(arguments); 
    }
 
    void dump()
    {
        _lock.lock();
        for(auto e : _elements)
            std::cout << e << std::endl;
        _lock.unlock();
    }
};
 
void func(container<int>& cont)
{
    cont.addrange(3, rand(), rand(), rand());
}
 
int main()
{
    srand((unsigned int)time(0));
 
    container<int> cont;
 
    std::thread t1(func, std::ref(cont));
    std::thread t2(func, std::ref(cont));
    std::thread t3(func, std::ref(cont));
 
    t1.join();
    t2.join();
    t3.join();
 
    cont.dump();
 
    return 0;
}
```

当你运行这个程序时，会进入死锁。原因：在 mutex 被释放前，容器尝试多次持有它，这显然不可能。这就是为什么引入 `std::recursive_mutex` ，它允许一个线程对 mutex 多重持有。允许的最大持有次数并不确定，但当达到上限时，线程锁会抛出 [`std::system_error`](http://en.cppreference.com/w/cpp/error/system_error) 错误。因此，要解决上面例子的错误，除了修改 `addrange` 令其不再调用 `lock` 和 `unlock` 之外，可以用 `std::recursive_mutex` 代替 `mutex`。

```cpp
template <typename T>
class container 
{
    std::recursive_mutex _lock;
    // ...
};
```

成功输出：

```
6334
18467
41
6334
18467
41
6334
18467
41
```

敏锐的读者可能注意到，每次调用 `func()` 输出的都是相同的数字。这是因为，seed 是线程局部量，调用 `srand()` 只会在主线程中初始化 seed，在其他工作线程中 seed 并未被初始化，所以每次得到的数字都是一样的。

手动加锁和解锁可能造成问题，比如忘记解锁或锁的次序出错，都会造成死锁。C++ 11 标准提供了若干类和函数来解决这个问题。封装类允许以 RAII 风格使用 mutex，在一个锁的生存周期内自动加锁和解锁。这些封装类包括：

- [lock_guard](http://en.cppreference.com/w/cpp/thread/lock_guardv)：当一个实例被创建时，会尝试持有 mutex （通过调用 `lock()`）；当实例销毁时，自动释放 mutex （通过调用 `unlock()`）。不允许拷贝。

- [unique_lock](http://en.cppreference.com/w/cpp/thread/unique_lock)：通用 mutex 封装类，与 `lock_guard` 不同，还支持延迟锁、计时锁、递归锁、移交锁的持有权，以及使用条件变量。不允许拷贝，但允许转移（move）。

借助这些封装类，可以把容器改写为：

```cpp
template <typename T>
class container 
{
    std::recursive_mutex _lock;
    std::vector<T> _elements;
public:
    void add(T element) 
    {
        std::lock_guard<std::recursive_mutex> locker(_lock);
        _elements.push_back(element);
    }
 
    void addrange(int num, ...)
    {
        va_list arguments;
 
        va_start(arguments, num);
 
        for (int i = 0; i < num; i++)
        {
            std::lock_guard<std::recursive_mutex> locker(_lock);
            add(va_arg(arguments, T));
        }
 
        va_end(arguments); 
    }
 
    void dump()
    {
        std::lock_guard<std::recursive_mutex> locker(_lock);
        for(auto e : _elements)
            std::cout << e << std::endl;
    }
};
```

读者可能会提出， `dump()` 方法不更改容器的状态，应该设为 const。但如果你添加 const 关键字，会得到如下编译错误：

```
‘std::lock_guard<_Mutex>::lock_guard(_Mutex &)' : cannot convert parameter 1 from ‘const std::recursive_mutex' to ‘std::recursive_mutex &'
```

一个 mutex （不管何种风格）必须被持有和释放，这意味着 `lock()` `unlock` 方法必被调用，这两个方法是 non-const 的。所以，逻辑上 `lock_guard` 的声明不能是 const （若该方法 为 const，则 mutex 也为 const）。这个问题的解决办法是，将 mutex 设为 `mutable`。`mutable` 允许由 const 方法更改 mutex 状态。不过，这种用法仅限于隐式的，或「元（meta）」状态——譬如，运算过的高速缓存、检索完成的数据，使得下次调用能瞬间完成；或者，改变像 mutex 之类的位元，仅仅作为一个对象的实际状态的补充。

```cpp
template <typename T>
class container 
{
   mutable std::recursive_mutex _lock;
   std::vector<T> _elements;
public:
   void dump() const
   {
      std::lock_guard<std::recursive_mutex> locker(_lock);
      for(auto e : _elements)
         std::cout << e << std::endl;
   }
};
```

这些封装类锁的构造函数可以通过重载的声明来指定锁的策略。可用的策略有：

- `defer_lock_t` 类型的 `defer_lock`：不持有 mutex

- `try_to_lock_t` 类型的  `try_to_lock`： 尝试持有 mutex 而不阻塞线程

- `adopt_lock_t` 类型的 `adopt_lock`：假定调用它的线程已持有 mutex

这些策略的声明方式如下：

```cpp
struct defer_lock_t { };
struct try_to_lock_t { };
struct adopt_lock_t { };
 
constexpr std::defer_lock_t defer_lock = std::defer_lock_t();
constexpr std::try_to_lock_t try_to_lock = std::try_to_lock_t();
constexpr std::adopt_lock_t adopt_lock = std::adopt_lock_t();
```

除了这些 mutex 封装类之外，标准库还提供了两个方法用于锁住一个或多个 mutex：

- [lock](http://en.cppreference.com/w/cpp/thread/lock)：锁住 mutex，通过一个避免了死锁的算法（通过调用 `lock()`，`try_lock()` 和 `unlock()` 实现）

- [try_lock](http://en.cppreference.com/w/cpp/thread/try_lock)：尝试通过调用 `try_lock()` 来调用多个 mutex，调用次序由 mutex 的指定次序而定

下面是一个死锁案例：有一个元素容器，以及一个 `exchange()` 函数用于互换两个容器里的某个元素。为了实现线程安全，这个函数通过一个和容器关联的 mutex，对这两个容器的访问进行同步。

```cpp
template <typename T>
class container 
{
public:
    std::mutex _lock;
    std::set<T> _elements;
 
    void add(T element) 
    {
        _elements.insert(element);
    }
 
    void remove(T element) 
    {
        _elements.erase(element);
    }
};
 
void exchange(container<int>& cont1, container<int>& cont2, int value)
{
    cont1._lock.lock();
    std::this_thread::sleep_for(std::chrono::seconds(1)); // <-- forces context switch to simulate the deadlock
    cont2._lock.lock();    
 
    cont1.remove(value);
    cont2.add(value);
 
    cont1._lock.unlock();
    cont2._lock.unlock();
}
```

假如这个函数在两个线程中被调用，在其中一个线程中，一个元素被移出容器 1 而加到容器 2；在另一个线程中，它被移出容器 2 而加到容器 1。这可能导致死锁——当一个线程刚持有第一个锁，程序马上切入另一个线程的时候。

```cpp
int main()
{
    srand((unsigned int)time(NULL));
 
    container<int> cont1; 
    cont1.add(1);
    cont1.add(2);
    cont1.add(3);
 
    container<int> cont2; 
    cont2.add(4);
    cont2.add(5);
    cont2.add(6);
 
    std::thread t1(exchange, std::ref(cont1), std::ref(cont2), 3);
    std::thread t2(exchange, std::ref(cont2), std::ref(cont1), 6);
 
    t1.join();
    t2.join();
 
    return 0;
}
```

要解决这个问题，可以使用 `std::lock`，保证所有的锁都以不会死锁的方式被持有：

```cpp
void exchange(container<int>& cont1, container<int>& cont2, int value)
{
    std::lock(cont1._lock, cont2._lock); 
 
    cont1.remove(value);
    cont2.add(value);
 
    cont1._lock.unlock();
    cont2._lock.unlock();
}
```

### 条件变量

C++ 11 提供的另一个同步机制是条件变量，用于阻塞一个或多个线程，直到接收到另一个线程的通知信号，或暂停信号，或[伪唤醒](https://en.wikipedia.org/wiki/Spurious_wakeup)信号。在 `<condition_variable>` 头文件里，有两个风格的条件变量实现：

- [condition_variable](http://en.cppreference.com/w/cpp/thread/condition_variable)：所有需要等待这个条件变量的线程，必须先持有一个 `std::unique_lock`

- [condition_variable_any](http://en.cppreference.com/w/cpp/thread/condition_variable_any)：更通用的实现，任何满足锁的基本条件（提供 `lock()` 和 `unlock()` 功能）的类型都可以使用；在性能和系统资源占用方面可能消耗更多，因而只有在它的灵活性成为必需的情况下才应优先使用

条件变量的工作机制如下：

- 至少有一个线程在等待某个条件成立。等待的线程必须先持有一个 `unique_lock` 锁。这个锁被传递给 `wait()` 方法，这会释放 mutex，阻塞线程直至条件变量收到通知信号。当收到通知信号，线程唤醒，重新持有锁。

- 至少有一个线程在发送条件成立的通知信号。信号的发送可以用 [`notify_one()`](http://en.cppreference.com/w/cpp/thread/condition_variable/notify_one) 方法， 只解锁任意一个正在等待通知信号的线程，也可以用 [`notify_all()`](http://en.cppreference.com/w/cpp/thread/condition_variable/notify_all) 方法， 解锁所有等待条件成立信号的线程。

- 在多核处理器系统上，由于使条件唤醒完全可预测的某些复杂机制的存在，可能发生伪唤醒，即一个线程在没有别的线程发送通知信号时也会唤醒。因而，当线程唤醒时，检查条件是否成立是必要的。而且，伪唤醒可能多次发生，所以条件检查要在一个循环里进行。


下面的代码展示使用条件变量进行线程同步的实例： 几个工作员线程在运行过程中会产生错误，他们将错误码存在一个队列里。一个记录员线程处理这些错误码，将错误码从记录队列里取出并打印出来。工作员会在发生错误时，给记录员发送信号。记录员则等待条件变量的通知信号。为了避免伪唤醒，等待工作放在一个检查布尔值的循环内。

```cpp
#include <thread>
#include <mutex>
#include <condition_variable>
#include <iostream>
#include <queue>
#include <random>

std::mutex              g_lockprint;
std::mutex              g_lockqueue;
std::condition_variable g_queuecheck;
std::queue<int>         g_codes;
bool                    g_done;
bool                    g_notified;

void workerfunc(int id, std::mt19937& generator)
{
    // print a starting message
    {
        std::unique_lock<std::mutex> locker(g_lockprint);
        std::cout << "[worker " << id << "]\trunning..." << std::endl;
    }

    // simulate work
    std::this_thread::sleep_for(std::chrono::seconds(1 + generator() % 5));

    // simulate error
    int errorcode = id*100+1;
    {
        std::unique_lock<std::mutex> locker(g_lockprint);
        std::cout  << "[worker " << id << "]\tan error occurred: " << errorcode << std::endl;
    }

    // notify error to be logged
    {
        std::unique_lock<std::mutex> locker(g_lockqueue);
        g_codes.push(errorcode);
        g_notified = true;
        g_queuecheck.notify_one();
    }
}

void loggerfunc()
{
    // print a starting message
    {
        std::unique_lock<std::mutex> locker(g_lockprint);
        std::cout << "[logger]\trunning..." << std::endl;
    }

    // loop until end is signaled
    while(!g_done)
    {
        std::unique_lock<std::mutex> locker(g_lockqueue);

        while(!g_notified) // used to avoid spurious wakeups 
        {
            g_queuecheck.wait(locker);
        }

        // if there are error codes in the queue process them
        while(!g_codes.empty())
        {
            std::unique_lock<std::mutex> locker(g_lockprint);
            std::cout << "[logger]\tprocessing error:  " << g_codes.front()  << std::endl;
            g_codes.pop();
        }

        g_notified = false;
    }
}

int main()
{
    // initialize a random generator
    std::mt19937 generator((unsigned int)std::chrono::system_clock::now().time_since_epoch().count());

    // start the logger
    std::thread loggerthread(loggerfunc);

    // start the working threads
    std::vector<std::thread> threads;
    for(int i = 0; i < 5; ++i)
    {
        threads.push_back(std::thread(workerfunc, i+1, std::ref(generator)));
    }

    // work for the workers to finish
    for(auto& t : threads)
        t.join();

    // notify the logger to finish and wait for it
    g_done = true;
    loggerthread.join();

    return 0;
}
```

运行这个程序，输出如下（注意这个输出在每次运行下都会改变，因为每个工作员线程的工作和休眠的时间间隔是任意的）：

```
[logger]        running...
[worker 1]      running...
[worker 2]      running...
[worker 3]      running...
[worker 4]      running...
[worker 5]      running...
[worker 1]      an error occurred: 101
[worker 2]      an error occurred: 201
[logger]        processing error:  101
[logger]        processing error:  201
[worker 5]      an error occurred: 501
[logger]        processing error:  501
[worker 3]      an error occurred: 301
[worker 4]      an error occurred: 401
[logger]        processing error:  301
[logger]        processing error:  401
```

上面的 `wait()` 有两个重载：

- 其中一个只需要传入一个 `unique_lock`；这个重载方法释放锁，阻塞线程并将其添加到一个等待该条件变量的线程队列里；该线程在收到条件变量通知信号或伪唤醒时唤醒，这时锁被重新持有，函数返回。

- 另外一个在 `unique_lock` 之外，还接收一个谓词（predicate），循环直至其返回 false；这个重载可用于避免伪唤醒，其功能类似于：

    ```cpp
    while(!predicate()) 
        wait(lock);
    ```

于是，上面例子中布尔值 `g_notified` 可以不用，而代之以 `wait` 的接收谓词的重载，用于确认状态队列的状态（是否为空）：

```cpp
void workerfunc(int id, std::mt19937& generator)
{
    // print a starting message
    {
        std::unique_lock<std::mutex> locker(g_lockprint);
        std::cout << "[worker " << id << "]\trunning..." << std::endl;
    }

    // simulate work
    std::this_thread::sleep_for(std::chrono::seconds(1 + generator() % 5));

    // simulate error
    int errorcode = id*100+1;
    {
        std::unique_lock<std::mutex> locker(g_lockprint);
        std::cout << "[worker " << id << "]\tan error occurred: " << errorcode << std::endl;
    }

    // notify error to be logged
    {
        std::unique_lock<std::mutex> locker(g_lockqueue);
        g_codes.push(errorcode);
        g_queuecheck.notify_one();
    }
}

void loggerfunc()
{
    // print a starting message
    {
        std::unique_lock<std::mutex> locker(g_lockprint);
        std::cout << "[logger]\trunning..." << std::endl;
    }

    // loop until end is signaled
    while(!g_done)
    {
        std::unique_lock<std::mutex> locker(g_lockqueue);

        g_queuecheck.wait(locker, [&](){return !g_codes.empty();});

        // if there are error codes in the queue process them
        while(!g_codes.empty())
        {
            std::unique_lock<std::mutex> locker(g_lockprint);
            std::cout << "[logger]\tprocessing error:  " << g_codes.front() << std::endl;
            g_codes.pop();
        }
    }
}
```

除了可重载的 `wait()`，还有另外两个等待方法，都有类似的接收谓词以避免伪唤醒的重载方法：

- [wait_for](http://en.cppreference.com/w/cpp/thread/condition_variable/wait_for)：阻塞线程，直至收到条件变量通知信号，或指定时间段已过去。

- [wait_until](http://en.cppreference.com/w/cpp/thread/condition_variable/wait_until)：阻塞线程，直到收到条件变量通知信号，或指定时间点已达到。

这两个方法如果不传入谓词，会返回一个 [`cv_status`](http://en.cppreference.com/w/cpp/thread/cv_status)，告知是到达设定时间还是线程因条件变量通知信号或伪唤醒而唤醒。

标准库还提供了 [`notify_all_at_thread_exit`](http://en.cppreference.com/w/cpp/thread/notify_all_at_thread_exit) 方法，实现了通知其他线程某个给定线程已经结束，以及销毁所有 `thread_local` 实例的机制。引入这个方法的原因是，在使用 `thread_local` 时， 等待一些通过非 `join()` 机制引入的线程可能造成错误行为，因为在等待的线程恢复或可能结束之后，他们的析构方法可能还在被调用（参看 [N3070](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2010/n3070.html) 和 [N2880](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2880.html)）。特别的，对这个函数的一个调用，必须发生在线程刚好退出之前。下面是一个 `notify_all_at_thread_exit` 和 `condition_variable` 搭配使用来同步两个线程的实例：

```cpp
std::mutex              g_lockprint;
std::mutex              g_lock;
std::condition_variable g_signal;
bool                    g_done;

void workerfunc(std::mt19937& generator)
{
   {
      std::unique_lock<std::mutex> locker(g_lockprint);
      std::cout << "worker running..." << std::endl;
   }

   std::this_thread::sleep_for(std::chrono::seconds(1 + generator() % 5));

   {
      std::unique_lock<std::mutex> locker(g_lockprint);
      std::cout << "worker finished..." << std::endl;
   }

   std::unique_lock<std::mutex> lock(g_lock);
   g_done = true;
   std::notify_all_at_thread_exit(g_signal, std::move(lock));
}

int main()
{
   // initialize a random generator
   std::mt19937 generator((unsigned int)std::chrono::system_clock::now().time_since_epoch().count());

   std::cout << "main running..." << std::endl;

   std::thread worker(workerfunc, std::ref(generator));
   worker.detach();

   std::cout << "main crunching..." << std::endl;

   std::this_thread::sleep_for(std::chrono::seconds(1 + generator() % 5));

   {
      std::unique_lock<std::mutex> locker(g_lockprint);
      std::cout << "main waiting for worker..." << std::endl;
   }

   std::unique_lock<std::mutex> lock(g_lock);
   while(!g_done) // avoid spurious wake-ups
      g_signal.wait(lock);

   std::cout << "main finished..." << std::endl;

   return 0;
}
```

如果 worker 在主线程之前结束，输出如下：

```
main running...
worker running...
main crunching...
worker finished...
main waiting for worker...
main finished...
```

如果主线程在 worker 线程之前结束，输出如下：

```
main running...
worker running...
main crunching...
main waiting for worker...
worker finished...
main finished...
```

### 小结

C++ 11 允许开发者们以标准的、不依赖于平台的方式编写多线程程序。这篇文章概述了标准库对于线程和同步操作机制的支持。`<thread>`头文件提供代表操作线程 `thread` 类和配套辅助方法。`<mutex>` 头文件提供几种 mutex 互斥锁及封装类，提供多线程同步访问机制。`<condition_variable>` 提供两种条件变量的实现，支持阻塞一个或多个线程直至接收到另一个线程发送的通知信号，或到达设定时间，或发生伪唤醒。建议读者对相关话题进行拓展阅读。

__授权证书__

本文及附加代码和资料，采用 [The Code Project Open License (CPOL)](http://www.codeproject.com/info/cpol10.aspx)。
