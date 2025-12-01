# C++ 标准库 <mutex>

在多线程编程中，确保数据的一致性和线程安全是至关重要的。

C++ 标准库提供了一套丰富的同步原语，用于控制对共享资源的访问。

C++ 标准库中的 `<mutex>`​ 头文件提供了一组工具，用于在多线程程序中实现线程间的同步和互斥。

​`<mutex>`​ 头文件是 C++11 引入的，它包含了用于互斥锁（mutex）的类和函数。互斥锁是一种同步机制，用于防止多个线程同时访问共享资源。

互斥锁（Mutex）是一个用于控制对共享资源访问的同步原语。当一个线程需要访问共享资源时，它会尝试锁定互斥锁。如果互斥锁已经被其他线程锁定，请求线程将被阻塞，直到互斥锁被释放。

## 基本语法

在 C++ 中，`<mutex>`​ 头文件提供了以下主要类：

* ​`std::mutex`​：基本的互斥锁。
* ​`std::recursive_mutex`​：递归互斥锁，允许同一个线程多次锁定。
* ​`std::timed_mutex`​：具有超时功能的互斥锁。
* ​`std::recursive_timed_mutex`​：具有超时功能的递归互斥锁。

## 实例

### 1. 使用 `std::mutex`​

下面是一个简单的示例，展示了如何在 C++ 中使用 `std::mutex`​ 来同步对共享资源的访问。

## 实例

#include <iostream>  
#include <thread>  
#include <mutex>

std::mutex mtx; // 全局互斥锁  
int shared\_resource = 0;

void increment() {  
    for (int i = 0; i < 10000; ++i) {  
        mtx.lock(); // 锁定互斥锁  
        ++shared\_resource;  
        mtx.unlock(); // 解锁互斥锁  
    }  
}

int main() {  
    std::thread t1(increment);  
    std::thread t2(increment);

    t1.join();  
    t2.join();

    std::cout << "Final value of shared_resource: " << shared\_resource << std::endl;  
    return 0;  
}

输出结果：

```
Final value of shared_resource: 20000
```

### 2. 使用 `std::recursive_mutex`​

递归互斥锁允许同一个线程多次锁定同一个互斥锁。下面是一个使用 `std::recursive_mutex`​ 的示例。

## 实例

#include <iostream>  
#include <thread>  
#include <mutex>

std::recursive_mutex rmtx; // 创建一个递归 mutex 对象  
int shared\_resource = 0; // 共享资源

// 递归函数  
void recursive\_increment(int count) {  
    if (count <= 0) return;

    std::lock_guard[std::recursive_mutex](std::recursive_mutex) lock(rmtx); // 上锁，确保线程安全  
    ++shared\_resource;  
    std::cout << "Incremented shared_resource to " << shared\_resource << " (count = " << count << ")" << std::endl;

    // 递归调用  
    recursive\_increment(count - 1);  
}

int main() {  
    std::thread t1(recursive\_increment, 3); // 线程 t1 执行 recursive_increment(3)  
    std::thread t2(recursive\_increment, 3); // 线程 t2 执行 recursive_increment(3)

    t1.join(); // 等待线程 t1 完成  
    t2.join(); // 等待线程 t2 完成

    std::cout << "Final value of shared_resource: " << shared\_resource << std::endl;

    return 0;  
}

输出结果：

```
Incremented shared_resource to 1 (count = 3)
Incremented shared_resource to 2 (count = 2)
Incremented shared_resource to 3 (count = 1)
Incremented shared_resource to 4 (count = 0)
Incremented shared_resource to 5 (count = 3)
Incremented shared_resource to 6 (count = 2)
Incremented shared_resource to 7 (count = 1)
Incremented shared_resource to 8 (count = 0)
Final value of shared_resource: 8
```

**代码解析：**

1. **创建** **​`std::recursive_mutex`​**​ **对象**：`std::recursive_mutex rmtx;`​ 是一个递归互斥量，允许同一线程多次获得锁。
2. **共享资源**：`int shared_resource = 0;`​ 是多个线程共同访问的资源。
3. **递归函数** **​`recursive_increment`​**​：

    * 使用 `std::lock_guard[std::recursive_mutex](std::recursive_mutex)`​ 对 `rmtx`​ 上锁。由于使用的是递归互斥量，同一个线程可以多次获得锁。
    * 通过 `++shared_resource`​ 增加共享资源的值。
    * 函数自身递归调用，演示了递归函数在多次锁定的情况下如何安全地工作。
4. **创建和运行线程**：

    * ​`std::thread t1(recursive_increment, 3);`​ 和 `std::thread t2(recursive_increment, 3);`​ 分别创建两个线程，它们都会执行 `recursive_increment(3)`​。
    * ​`t1.join()`​ 和 `t2.join()`​ 等待两个线程执行完毕。

## 注意事项

* 确保在每次锁定互斥锁后，都进行解锁操作，以避免死锁。
* 使用 `std::lock_guard`​ 或 `std::unique_lock`​ 可以自动管理互斥锁的锁定和解锁，减少出错的可能性。
* 避免在持有互斥锁的情况下调用可能抛出异常的函数，因为这可能导致死锁。

​`<mutex>`​ 是 C++ 标准库中一个非常重要的头文件，它为多线程编程提供了基本的同步机制。通过使用互斥锁，我们可以确保对共享资源的访问是安全的，从而避免数据竞争和不一致的问题。

---

## 更多说明

### 1. std::mutex

提供基本的互斥量，确保在同一时刻只有一个线程可以访问共享资源。

## 实例

#include <mutex>

std::mutex mtx;

void thread\_function() {  
    std::lock_guard[std::mutex](std::mutex) lock(mtx);  
    // 访问共享资源  
}

### 2. std::recursive\_mutex

允许同一线程多次获得锁，而不会造成死锁，这对递归函数特别有用。

## 实例

#include <mutex>

std::recursive_mutex rmtx;

void recursive\_function(int count) {  
    if (count <= 0) return;  
    std::lock_guard[std::recursive_mutex](std::recursive_mutex) lock(rmtx);  
    // 递归调用  
    recursive\_function(count - 1);  
}

### 3. std::timed\_mutex

提供定时的互斥量，可以在尝试获得锁时设置超时时间。

## 实例

#include <mutex>  
#include <chrono>

std::timed_mutex tm;

void try\_lock\_for\_example() {  
    if (tm.try_lock_for(std::chrono::seconds(1))) {  
        // 成功获得锁  
        tm.unlock();  
    } else {  
        // 锁获取失败  
    }  
}

### 4. std::recursive\_timed\_mutex

继承自 std::timed\_mutex，允许同一线程多次获得锁，同时支持定时功能。

## 实例

#include <mutex>  
#include <chrono>

std::recursive_timed_mutex rtm;

void recursive\_timed\_function(int count) {  
    if (count <= 0) return;  
    if (rtm.try_lock_for(std::chrono::seconds(1))) {  
        // 成功获得锁  
        recursive\_timed\_function(count - 1);  
        rtm.unlock();  
    } else {  
        // 锁获取失败  
    }  
}

### 5. std::lock\_guard

一种自动管理 std::mutex 锁的封装器，使用 RAII 风格，确保在作用域结束时自动释放锁。

## 实例

#include <mutex>

std::mutex mtx;

void function() {  
    std::lock_guard[std::mutex](std::mutex) lock(mtx);  
    // 访问共享资源  
}

### 6. std::unique\_lock

提供比 std::lock\_guard 更灵活的锁管理，可以手动释放和重新获得锁，还支持定时锁定。

## 实例

#include <mutex>  
#include <chrono>

std::mutex mtx;

void function() {  
    std::unique_lock[std::mutex](std::mutex) lock(mtx);  
    // 访问共享资源

    // 可以手动释放锁  
    lock.unlock();

    // 可以重新获得锁  
    lock.lock();

    // 可以进行定时锁定  
    if (lock.try_lock_for(std::chrono::seconds(1))) {  
        // 成功获得锁  
    }  
}

### 7. std::adopt\_lock\_t

标志类型，用于指定 std::unique\_lock 采用已有的锁。

## 实例

std::mutex mtx;  
std::unique_lock[std::mutex](std::mutex) lock(mtx, std::adopt_lock);

### 8. std::defer\_lock\_t

功能：标志类型，用于延迟锁定，初始化时不锁定。

## 实例

std::mutex mtx;  
std::unique_lock[std::mutex](std::mutex) lock(mtx, std::defer_lock);  
// 可以在之后的某个时刻调用 lock.lock()

### 9. std::try\_to\_lock\_t

标志类型，用于尝试锁定而不阻塞。

## 实例

std::mutex mtx1, mtx2;  
std::unique_lock[std::mutex](std::mutex) lock1(mtx1, std::try_to_lock);  
std::unique_lock[std::mutex](std::mutex) lock2(mtx2, std::try_to_lock);

if (lock1 && lock2) {  
    // 成功获得两个锁  
}
