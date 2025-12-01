# C++ 标准库 <chrono>

C++11 引入了 `<chrono>`​ 库，这是一个用于处理时间和日期的库。它提供了一套丰富的工具来测量时间间隔、执行时间点的计算以及处理日期和时间。`<chrono>`​ 库是 C++ 标准库中处理时间相关操作的核心部分。

## 基本概念

### 时间点（Time Points）

时间点表示一个特定的时间点，通常与某个特定的时钟相关联。

### 持续时间（Durations）

持续时间表示两个时间点之间的时间间隔。

### 时钟（Clocks）

时钟是时间点和持续时间的来源。C++ 提供了几种不同的时钟，例如系统时钟、高分辨率时钟等。

## 基本语法

### 包含头文件

在使用 `<chrono>`​ 库之前，需要包含相应的头文件：

```
#include <chrono>
```

### 使用时间点

```
auto now = std::chrono::system_clock::now();
```

### 使用持续时间

```
auto duration = std::chrono::seconds(5);
```

### 计算时间点

```
auto future_time = now + duration;
```

## 实例

### 测量函数执行时间

下面是一个使用 `<chrono>`​ 库测量函数执行时间的简单示例：

## 实例

#include <iostream>  
#include <chrono>

void someFunction() {  
    // 模拟一些操作  
    std::this_thread::sleep_for(std::chrono::seconds(1));  
}

int main() {  
    auto start = std::chrono::high_resolution_clock::now();

    someFunction();

    auto end = std::chrono::high_resolution_clock::now();

    auto duration = std::chrono::duration_cast[std::chrono::milliseconds](std::chrono::milliseconds)(end - start);  
    std::cout << "Function took " << duration.count() << " milliseconds to execute." << std::endl;

    return 0;  
}

输出结果：

```
Function took 1000 milliseconds to execute.
```

### 处理日期和时间

​`<chrono>`​ 库也可以用来处理日期和时间。下面是一个使用 `std::chrono::system_clock`​ 和 `std::chrono::time_point`​ 来获取当前日期和时间的示例：

## 实例

#include <iostream>  
#include <chrono>  
#include <ctime>

int main() {  
    auto now = std::chrono::system_clock::now();  
    std::time_t now\_c = std::chrono::system_clock::to_time_t(now);

    std::cout << "Current date and time: " << std::ctime(&now\_c);

    return 0;  
}

输出结果：

```
Current date and time: Fri Mar 11 12:34:56 2022
```

## 高级用法

### 使用不同的时钟

C++ 提供了多种时钟，例如：

* ​`std::chrono::system_clock`​：系统时钟，通常与系统时间同步。
* ​`std::chrono::steady_clock`​：单调时钟，不会受到系统时间变化的影响。
* ​`std::chrono::high_resolution_clock`​：提供最高分辨率的时钟。

### 格式化日期和时间

可以使用 `<iomanip>`​ 和 `<ctime>`​ 来格式化日期和时间：

## 实例

#include <iostream>  
#include <iomanip>  
#include <chrono>  
#include <ctime>

int main() {  
    auto now = std::chrono::system_clock::now();  
    std::time_t now\_c = std::chrono::system_clock::to_time_t(now);

    std::cout << std::put_time(std::localtime(&now\_c), "%Y-%m-%d %H:%M:%S");

    return 0;  
}

输出结果：

```
2022-03-11 12:34:56
```
