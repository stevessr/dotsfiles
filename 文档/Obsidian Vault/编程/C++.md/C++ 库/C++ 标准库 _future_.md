# C++ 标准库 <future>

C++11 引入了 `<future>`​ 头文件，它提供了一种异步编程的机制，允许程序在等待某个操作完成时继续执行其他任务。`<future>`​ 库是 C++ 标准库中并发编程的一部分，它允许程序员以一种更简洁和安全的方式处理异步操作。

​`<future>`​ 库中定义了几个关键的类型：

* ​`std::future`​：表示异步操作的结果，可以查询操作的状态，获取结果或等待操作完成。
* ​`std::promise`​：用于与 `std::future`​ 配对，用于设置异步操作的结果。
* ​`std::packaged_task`​：封装一个函数或可调用对象，使其可以作为异步任务执行。

### std::promise

​`std::promise`​ 用于设置异步操作的结果。它与 `std::future`​ 配对使用。

## 实例

#include <iostream>  
#include <future>

int main() {  
    std::promise<int> prom;  
    std::future<int> fut = prom.get_future();

    // 在另一个线程中设置结果  
    std::thread t([prom]() {  
        prom.set_value(10);  
    });

    // 等待结果  
    std::cout << "Future value: " << fut.get() << std::endl;

    t.join();  
    return 0;  
}

输出结果：

```
Future value: 10
```

### std::packaged\_task

​`std::packaged_task`​ 封装一个函数或可调用对象，使其可以作为异步任务执行。

## 实例

#include <iostream>  
#include <future>  
#include <cmath>

int compute\_square\_root(double x) {  
    return std::sqrt(x);  
}

int main() {  
    std::packaged_task<double(double)> task(compute\_square\_root);  
    std::future<double> result = task.get_future();  
    std::thread th(std::move(task), 9.0);

    std::cout << "Result: " << result.get() << std::endl;  
    th.join();  
    return 0;  
}

输出结果：

```
Result: 3
```

### std::async

​`std::async`​ 是一个方便的函数，用于启动异步任务。它可以立即返回一个 `std::future`​ 对象。

## 实例

#include <iostream>  
#include <future>

int main() {  
    std::future<int> fut = std::async(std::launch::async, [](int x) {  
        return x * x;  
    }, 5);

    std::cout << "Result: " << fut.get() << std::endl;  
    return 0;  
}

输出结果：

```
Result: 25
```

## 异常处理

当异步操作抛出异常时，`std::future`​ 会捕获这个异常，并且可以通过调用 `.get()`​ 方法来重新抛出它。

## 实例

#include <iostream>  
#include <future>

void throw\_exception() {  
    throw std::runtime_error("Exception thrown");  
}

int main() {  
    std::future<void> fut = std::async(throw\_exception);

    try {  
        fut.get();  
    } catch (const std::exception& e) {  
        std::cout << "Caught exception: " << e.what() << std::endl;  
    }  
    return 0;  
}

输出结果：

```
Caught exception: Exception thrown
```

​`<future>`​ 库为 C++ 程序员提供了一种简单而强大的异步编程方式。通过使用 `std::promise`​、`std::packaged_task`​ 和 `std::async`​，我们可以轻松地在 C++ 程序中实现并发和异步操作。同时，异常处理机制也确保了程序的健壮性。
