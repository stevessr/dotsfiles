# C++ 标准库 <iterator>

C++ 标准库中的 `<iterator>`​ 头文件提供了一组工具，用于遍历容器中的元素。迭代器是 C++ 标准模板库（STL）中的核心概念之一，它允许程序员以统一的方式访问容器中的元素，而不需要关心容器的具体实现细节。

迭代器是一个对象，它提供了一种方法来遍历容器中的元素。迭代器可以被视为指向容器中元素的指针，但它比指针更加灵活和强大。迭代器可以用于访问、修改容器中的元素，并且可以与 STL 算法一起使用。

迭代器主要分为以下几类：

1. **输入迭代器（Input Iterator）** ：只能进行单次读取操作，不能进行写入操作。
2. **输出迭代器（Output Iterator）** ：只能进行单次写入操作，不能进行读取操作。
3. **正向迭代器（Forward Iterator）** ：可以进行读取和写入操作，并且可以向前移动。
4. **双向迭代器（Bidirectional Iterator）** ：除了可以进行正向迭代器的所有操作外，还可以向后移动。
5. **随机访问迭代器（Random Access Iterator）** ：除了可以进行双向迭代器的所有操作外，还可以进行随机访问，例如通过下标访问元素。

## 迭代器的语法

迭代器的语法通常如下：

```
#include <iterator>

// 使用迭代器遍历容器
for (ContainerType::iterator it = container.begin(); it != container.end(); ++it) {
    // 访问元素 *it
}
```

## 实例

下面是一个使用 `<iterator>`​ 头文件和迭代器遍历 `std::vector`​ 的示例：

## 实例

#include <iostream>  
#include <vector>  
#include <iterator>

int main() {  
    // 创建一个 vector 容器并初始化  
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 使用迭代器遍历 vector  
    for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it) {  
        std::cout << *it << " ";  
    }  
    std::cout << std::endl;

    // 使用 auto 关键字简化迭代器类型  
    for (auto it = vec.begin(); it != vec.end(); ++it) {  
        std::cout << *it << " ";  
    }  
    std::cout << std::endl;

    // 使用 C++11 范围 for 循环  
    for (int elem : vec) {  
        std::cout << elem << " ";  
    }  
    std::cout << std::endl;

    return 0;  
}

输出结果:

```
1 2 3 4 5 
1 2 3 4 5 
1 2 3 4 5 
```

通过使用 `<iterator>`​ 头文件，我们可以方便地遍历 C++ STL 容器中的元素。迭代器提供了一种统一的接口，使得我们可以在不同的容器之间切换，而不需要改变遍历的代码。这大大提高了代码的可重用性和可维护性。

对于初学者来说，理解迭代器的概念和使用方式是非常重要的，因为它们是 C++ STL 的基础。希望这篇文章能帮助你更好地理解迭代器，并在你的 C++ 编程中有效地使用它们。
