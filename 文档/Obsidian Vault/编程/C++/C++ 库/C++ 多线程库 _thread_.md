# C++ 多线程库 <thread>

C++11 引入了多线程支持，通过 `<thread>`​ 库，开发者可以轻松地在程序中实现并行处理。

本文将将介绍 `<thread>`​ 库的基本概念、定义、语法以及如何使用它来创建和管理线程。

线程是程序执行的最小单元，是操作系统能够进行运算调度的最小单位。

在多线程程序中，多个线程可以并行执行，提高程序的执行效率。

### C++ `<thread>`​ 库概述

​`<thread>`​ 库是 C++ 标准库的一部分，提供了创建和管理线程的基本功能，它包括以下几个关键组件：

* ​`std::thread`​：表示一个线程，可以创建、启动、等待和销毁线程。
* ​`std::this_thread`​：提供了一些静态成员函数，用于操作当前线程。
* ​`std::thread::id`​：线程的唯一标识符。

### 创建线程

要创建一个线程，你需要实例化 `std::thread`​ 类，并传递一个可调用对象（函数、lambda 表达式或对象的成员函数）作为参数。

## 实例

#include <iostream>  
#include <thread>

void print\_id(int id) {  
    std::cout << "ID: " << id << ", Thread ID: " << std::this_thread::get_id() << std::endl;  
}

int main() {  
    std::thread t1(print\_id, 1);  
    std::thread t2(print\_id, 2);  
}

### 启动线程

创建 `std::thread`​ 对象后，线程会立即开始执行，你可以调用 `join()`​ 方法来等待线程完成。

```
t1.join();
t2.join();
```

### 等待线程完成

​`join()`​ 方法会阻塞当前线程，直到被调用的线程完成执行。

### 销毁线程

当线程执行完毕后，你可以使用 `detach()`​ 方法来分离线程，或者让 `std::thread`​ 对象超出作用域自动销毁。

```
t1.detach(); // 线程将继续运行，但无法再被 join 或 detach
```

## 实例：使用 `<thread>`​ 创建并行计算

下面是一个使用 `<thread>`​ 库实现的并行计算实例，计算两个数的和。

## 实例

#include <iostream>  
#include <thread>

int sum = 0;

void add(int a, int b) {  
    sum += a + b;  
}

int main() {  
    int a = 5;  
    int b = 10;

    std::thread t1(add, a, b);  
    std::thread t2(add, a, b);

    t1.join();  
    t2.join();

    std::cout << "Sum: " << sum << std::endl; // 输出结果：Sum: 30  
}

输出结果为：

```
Sum: 30
```

以下实例我们将创建两个线程，每个线程都会执行一个简单的函数，该函数打印一个消息并休眠一段时间：

## 实例

#include <iostream>  
#include <thread>  
#include <chrono>

// 简单的函数，在线程中执行  
void print\_message(const std::string& message, int delay) {  
    std::this_thread::sleep_for(std::chrono::milliseconds(delay));  
    std::cout << message << std::endl;  
}

int main() {  
    // 创建两个线程，执行 print_message 函数  
    std::thread t1(print\_message, "Hello from thread 1", 1000);  
    std::thread t2(print\_message, "Hello from thread 2", 500);

    // 等待线程 t1 完成  
    if (t1.joinable()) {  
        t1.join();  
    }

    // 等待线程 t2 完成  
    if (t2.joinable()) {  
        t2.join();  
    }

    std::cout << "Main thread finished." << std::endl;

    return 0;  
}

输出结果为：

```
Hello from thread 2
Hello from thread 1
Main thread finished.
```

### 注意事项

* 线程安全：在多线程环境中，共享资源需要同步访问，以避免数据竞争。
* 线程生命周期：确保在线程执行完毕后正确地处理线程对象，避免资源泄露。

---

## 类和函数

​`<thread>`​ 库包含了一系列的类和函数，用于创建、管理和同步线程。

以下是对 C++ `<thread>`​ 库的详细介绍:

### 主要组件

* **std::thread**
* **std::mutex**
* **std::lock_guard**
* **std::unique_lock**
* **std::condition_variable**
* **std::future 和 std::promise**
* **std::async**

### std::thread

std::thread 类用于创建和管理线程。

## 实例

#include <iostream>  
#include <thread>

void print\_hello() {  
    std::cout << "Hello from thread!" << std::endl;  
}

int main() {  
    std::thread t(print\_hello);  
    t.join(); // 等待线程 t 结束  
    return 0;  
}

**重要方法**

* **join()** : 等待线程结束。
* **detach()** : 将线程置于后台运行，不再等待线程结束。
* **joinable()** : 检查线程是否可被 join 或 detach。

### std::mutex

std::mutex 类用于同步对共享资源的访问。

## 实例

#include <iostream>  
#include <thread>  
#include <mutex>

std::mutex mtx; // 创建一个全局 mutex 对象  
int shared\_resource = 0; // 共享资源

// 线程函数  
void increment() {  
    std::lock_guard[std::mutex](std::mutex) lock(mtx); // 上锁，保证线程安全  
    ++shared\_resource;  
    std::cout << "Incremented shared_resource to " << shared\_resource << std::endl;  
    // lock 在 lock_guard 离开作用域时自动释放  
}

int main() {  
    std::thread t1(increment);  
    std::thread t2(increment);

    t1.join(); // 等待线程 t1 完成  
    t2.join(); // 等待线程 t2 完成

    std::cout << "Final value of shared_resource: " << shared\_resource << std::endl;

    return 0;  
}

### std::lock\_guard

std::lock\_guard 是一个 RAII 风格的锁管理器，用于自动管理锁的生命周期。

## 实例

#include <iostream>  
#include <thread>  
#include <mutex>

std::mutex mtx;

void print\_thread\_id(int id) {  
    std::lock_guard[std::mutex](std::mutex) lock(mtx);  
    std::cout << "Thread ID: " << id << std::endl;  
}

int main() {  
    std::thread t1(print\_thread\_id, 1);  
    std::thread t2(print\_thread\_id, 2);  
    t1.join();  
    t2.join();  
    return 0;  
}

### std::unique\_lock

std::unique\_lock 提供了比 std::lock\_guard 更灵活的锁管理。

## 实例

#include <iostream>  
#include <thread>  
#include <mutex>

std::mutex mtx;

void print\_thread\_id(int id) {  
    std::unique_lock[std::mutex](std::mutex) lock(mtx);  
    std::cout << "Thread ID: " << id << std::endl;  
    lock.unlock(); // 可以手动解锁  
    // ... 其他操作  
}

int main() {  
    std::thread t1(print\_thread\_id, 1);  
    std::thread t2(print\_thread\_id, 2);  
    t1.join();  
    t2.join();  
    return 0;  
}

### std::condition\_variable

std::condition\_variable 用于线程间的等待和通知。

## 实例

#include <iostream>  
#include <thread>  
#include <mutex>  
#include <condition_variable>

std::mutex mtx;  
std::condition_variable cv;  
bool ready = false;

void print\_id(int id) {  
    std::unique_lock[std::mutex](std::mutex) lock(mtx);  
    cv.wait(lock, []{ return ready; });  
    std::cout << "Thread ID: " << id << std::endl;  
}

void set\_ready() {  
    std::unique_lock[std::mutex](std::mutex) lock(mtx);  
    ready = true;  
    cv.notify_all();  
}

int main() {  
    std::thread t1(print\_id, 1);  
    std::thread t2(print\_id, 2);

    std::this_thread::sleep_for(std::chrono::seconds(1));  
    set\_ready();

    t1.join();  
    t2.join();  
    return 0;  
}

### std::future 和 std::promise

std::future 和 std::promise 用于线程间的结果传递。

## 实例

#include <iostream>  
#include <thread>  
#include <future>

void calculate\_square(std::promise<int> && p, int x) {  
    p.set_value(x * x);  
}

int main() {  
    std::promise<int> p;  
    std::future<int> f = p.get_future();

    std::thread t(calculate\_square, std::move(p), 5);

    std::cout << "Square: " << f.get() << std::endl;

    t.join();  
    return 0;  
}

### std::async

std::async 用于启动异步任务，并返回一个 std::future。

## 实例

#include <iostream>  
#include <future>

int calculate\_square(int x) {  
    return x * x;  
}

int main() {  
    std::future<int> result = std::async(calculate\_square, 5);

    std::cout << "Square: " << result.get() << std::endl;

    return 0;  
}

C++ 的 `<thread>`​ 库为开发者提供了强大的多线程支持。通过本文的介绍，我们应该能够理解线程的基本概念，并学会如何使用 `<thread>`​ 库来创建和管理线程。在实际开发中，合理利用多线程可以显著提高程序的性能和响应速度。
