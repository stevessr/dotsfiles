# C++ 标准库 <utility>

C++ 标准库（Standard Template Library，STL）是 C++ 的核心组成部分，提供了一组丰富的工具和算法，以帮助开发者更高效地编写代码。

在 C++ 标准库中，`<utility>`​ 头文件包含了一些实用的工具类和函数，这些工具类和函数在编写高效、可读性强的代码时非常有用。

​`<utility>`​ 头文件定义了多种工具类和函数，它们主要用于简化编程任务，提高代码的可读性和可维护性，这些工具类和函数包括：

* ​`pair`​：一个包含两个元素的容器，通常用于存储和返回两个相关联的值。
* ​`make_pair`​：一个函数模板，用于创建 `pair`​ 对象。
* ​`swap`​：一个函数模板，用于交换两个对象的值。
* ​`forward`​ 和 `move`​：用于完美转发和移动语义的函数模板。

## 语法

### pair 类

​`pair`​ 是一个模板类，可以存储两个不同类型的值。其基本语法如下：

```
#include <utility>

std::pair<T1, T2> p;
```

这里，`T1`​ 和 `T2`​ 是两个不同的类型，`p`​ 是一个 `pair`​ 对象，包含两个元素：`first`​ 和 `second`​。

### make\_pair 函数

​`make_pair`​ 是一个函数模板，用于创建 `pair`​ 对象。其基本语法如下：

```
#include <utility>

auto p = std::make_pair(x, y);
```

这里，`x`​ 和 `y`​ 是两个值，`p`​ 是一个 `pair`​ 对象，其 `first`​ 成员存储 `x`​，`second`​ 成员存储 `y`​。

### swap 函数

​`swap`​ 是一个函数模板，用于交换两个对象的值。其基本语法如下：

```
#include <utility>

std::swap(a, b);
```

这里，`a`​ 和 `b`​ 是两个对象，`swap`​ 函数将它们的值进行交换。

## 实例

### 使用 pair 和 make\_pair

## 实例

#include <iostream>  
#include <utility>

int main() {  
    // 使用 make_pair 创建 pair 对象  
    auto p = std::make_pair(10, 20);

    // 输出 pair 对象的元素  
    std::cout << "First element: " << p.first << std::endl;  
    std::cout << "Second element: " << p.second << std::endl;

    return 0;  
}

输出结果：

```
First element: 10
Second element: 20
```

### 使用 swap 函数

## 实例

#include <iostream>  
#include <utility>

int main() {  
    int a = 5;  
    int b = 10;

    std::cout << "Before swap: a = " << a << ", b = " << b << std::endl;

    // 使用 swap 函数交换 a 和 b 的值  
    std::swap(a, b);

    std::cout << "After swap: a = " << a << ", b = " << b << std::endl;

    return 0;  
}

输出结果：

```
Before swap: a = 5, b = 10
After swap: a = 10, b = 5
```

### 使用 move 函数

## 实例

#include <iostream>  
#include <utility>  
#include <vector>

int main() {  
    std::vector<int> v1 = {1, 2, 3, 4, 5};  
    std::vector<int> v2 = std::move(v1);

    std::cout << "v1 size: " << v1.size() << std::endl; // v1 现在为空  
    std::cout << "v2 size: " << v2.size() << std::endl; // v2 拥有 v1 的元素

    return 0;  
}

### 使用 forward 函数

## 实例

#include <iostream>  
#include <utility>

void process(int& i) {  
    std::cout << "Lvalue reference to " << i << std::endl;  
}

void process(int&& i) {  
    std::cout << "Rvalue reference to " << i << std::endl;  
}

template <typename T>  
void forward\_example(T&& t) {  
    process(std::forward<T>(t));  
}

int main() {  
    int x = 10;  
    forward\_example(x);           // Lvalue reference  
    forward\_example(20);          // Rvalue reference  
    forward\_example(std::move(x));// Rvalue reference

    return 0;  
}

### std::declval 实例

## 实例

#include <iostream>  
#include <utility>  
#include <type_traits>

struct MyClass {  
    MyClass(int, double) {}  
};

template <typename T>  
void test() {  
    // 获取 T 的构造函数返回的类型，而不实际调用它  
    using R = decltype(std::declval<T>()(1, 1.0));  
    std::cout << std::is_same<R, MyClass>::value << std::endl;  
}

int main() {  
    test<MyClass>(); // 输出 1 (true)

    return 0;  
}

​`<utility>`​ 头文件是 C++ 标准库中一个非常有用的部分，它提供了多种工具类和函数，帮助开发者编写更简洁、更高效的代码。
