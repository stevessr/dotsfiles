# C++ 标准库 <atomic>

在多线程编程中，数据的同步和线程安全是一个重要的问题。

C++11 标准引入了 `<atomic>`​ 库，它提供了一组原子操作，用于保证在多线程环境下对单个数据的访问是原子的，即不可分割的。这可以避免数据竞争和保证线程安全。

原子操作是指在执行过程中不会被其他线程中断的操作。

​`<atomic>`​库中的原子类型提供了这样的操作，它们可以保证在多线程环境中对共享数据的访问是安全的。

## 语法

​`<atomic>`​库提供了多种原子类型，包括`atomic<bool>`​, `atomic<char>`​, `atomic<short>`​, `atomic<int>`​, `atomic<long>`​, `atomic<long long>`​, `atomic<wchar_t>`​, `atomic<char16_t>`​, `atomic<char32_t>`​, `atomic<unsigned char>`​, `atomic<unsigned short>`​, `atomic<unsigned>`​, `atomic<unsigned long>`​, `atomic<unsigned long long>`​, `atomic<float>`​, `atomic<double>`​, `atomic<long double>`​等。

### 基本操作

* ​`load()`​: 安全地读取原子变量的值。
* ​`store(value)`​: 安全地将值写入原子变量。
* ​`exchange(value)`​: 将原子变量的值替换为`value`​，并返回原子变量的旧值。
* ​`compare_exchange_weak(expected, desired)`​: 如果原子变量的当前值等于`expected`​，则将其设置为`desired`​，并返回`true`​。否则，将`expected`​设置为原子变量的当前值，并返回`false`​。
* ​`compare_exchange_strong(expected, desired)`​: 与`compare_exchange_weak`​类似，但循环直到成功。

## 实例

下面是一个使用`<atomic>`​库的简单示例，演示了如何在多线程环境中安全地更新一个共享计数器。

## 实例

#include <iostream>  
#include <atomic>  
#include <thread>

std::atomic<int> counter(0); // 初始化原子计数器

void increment() {  
    for (int i = 0; i < 10000; ++i) {  
        counter.fetch_add(1, std::memory_order_relaxed); // 原子增加  
    }  
}

int main() {  
    std::thread t1(increment);  
    std::thread t2(increment);

    t1.join();  
    t2.join();

    std::cout << "Final counter value: " << counter << std::endl; // 输出最终的计数器值

    return 0;  
}

运行上述程序，你将看到输出类似于：

```
Final counter value: 20000
```

这个输出表明两个线程成功地在没有数据竞争的情况下，各自增加了10000次计数器的值。

### 注意事项

* 使用`<atomic>`​库时，需要确保所有对共享数据的访问都是通过原子操作进行的，以避免数据竞争。
* 不同的原子操作有不同的内存顺序要求，`std::memory_order_relaxed`​是最低的内存顺序要求，但可能不保证操作的可见性。根据需要选择合适的内存顺序。
* 原子操作的性能开销通常比非原子操作要高，因此在单线程环境中，使用普通变量可能更高效。

通过使用`<atomic>`​库，C++程序员可以更容易地编写线程安全的代码，同时保持高性能。
