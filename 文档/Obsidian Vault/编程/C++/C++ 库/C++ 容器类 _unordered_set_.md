# C++ 容器类 <unordered_set>

在C++中，`<unordered_set>`​ 是标准模板库（STL）的一部分，提供了一种基于哈希表的容器，用于存储唯一的元素集合。

与 `set`​ 不同，`unordered_set`​ 不保证元素的排序，但通常提供更快的查找、插入和删除操作。

​`unordered_set`​ 是一个模板类，其定义如下：

```
#include <unordered_set>

std::unordered_set<Key, Hash = std::hash<Key>, Pred = std::equal_to<Key>, Alloc = std::allocator<Key>>
```

* ​`Key`​ 是存储在 `unordered_set`​ 中的元素类型。
* ​`Hash`​ 是一个函数或函数对象，用于生成元素的哈希值，默认为 `std::hash<Key>`​。
* ​`Pred`​ 是一个二元谓词，用于比较两个元素是否相等，默认为 `std::equal_to<Key>`​。
* ​`Alloc`​ 是分配器类型，用于管理内存分配，默认为 `std::allocator<Key>`​。

### 语法

以下是一些基本的 `unordered_set`​ 操作：

* **构造函数**：创建一个空的 `unordered_set`​。

  ```
  std::unordered_set<int> uset;
  ```
* **插入元素**：使用 `insert()`​ 方法。

  ```
  uset.insert(10);
  ```
* **查找元素**：使用 `find()`​ 方法。

  ```
  auto it = uset.find(10);
  if (it != uset.end()) {
    // 元素存在
  }
  ```
* **删除元素**：使用 `erase()`​ 方法。

  ```
  uset.erase(10);
  ```
* **大小和空检查**：使用 `size()`​ 和 `empty()`​ 方法。

  ```
  size_t size = uset.size();
  bool isEmpty = uset.empty();
  ```
* **清空容器**：使用 `clear()`​ 方法。

  ```
  uset.clear();
  ```

## 实例

下面是一个使用 `unordered_set`​ 的简单示例，包括输出结果。

## 实例

#include <iostream>  
#include <unordered_set>

int main() {  
    // 创建一个整数类型的 unordered_set  
    std::unordered_set<int> uset;

    // 插入元素  
    uset.insert(10);  
    uset.insert(20);  
    uset.insert(30);

    // 打印 unordered_set 中的元素  
    std::cout << "Elements in uset: ";  
    for (int elem : uset) {  
        std::cout << elem << " ";  
    }  
    std::cout << std::endl;

    // 查找元素  
    auto it = uset.find(20);  
    if (it != uset.end()) {  
        std::cout << "Element 20 found in uset." << std::endl;  
    } else {  
        std::cout << "Element 20 not found in uset." << std::endl;  
    }

    // 删除元素  
    uset.erase(20);  
    std::cout << "After erasing 20, elements in uset: ";  
    for (int elem : uset) {  
        std::cout << elem << " ";  
    }  
    std::cout << std::endl;

    // 检查大小和是否为空  
    std::cout << "Size of uset: " << uset.size() << std::endl;  
    std::cout << "Is uset empty? " << (uset.empty() ? "Yes" : "No") << std::endl;

    // 清空 unordered_set  
    uset.clear();  
    std::cout << "After clearing, is uset empty? " << (uset.empty() ? "Yes" : "No") << std::endl;

    return 0;  
}

输出结果:

```
Elements in uset: 10 20 30 
Element 20 found in uset.
After erasing 20, elements in uset: 10 30 
Size of uset: 2
Is uset empty? No
After clearing, is uset empty? Yes
```

​`unordered_set`​ 是一个非常有用的容器，特别适合于需要快速查找、插入和删除操作的场景，同时不需要元素的有序性。
