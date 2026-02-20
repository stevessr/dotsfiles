# C++ 标准库 <sstream>

在 C++ 编程中，处理字符串和数字之间的转换是一项常见的任务。

​`sstream`​ 是 C++ 标准库中的一个组件，它提供了一种方便的方式来处理字符串流（可以像处理流一样处理字符串）。

​`<sstream>`​ 允许你将字符串当作输入/输出流来使用，这使得从字符串中读取数据或将数据写入字符串变得非常简单。

### 定义

​`sstream`​是 C++ 标准库中的一个命名空间，它包含了几个类，用于处理字符串流，这些类包括：

* ​`istringstream`​：用于从字符串中读取数据。
* ​`ostringstream`​：用于将数据写入字符串。
* ​`stringstream`​：是`istringstream`​和`ostringstream`​的组合，可以同时进行读取和写入操作。

### 语法

使用`sstream`​的基本语法如下：

```
#include <sstream>

// 使用istringstream
std::istringstream iss("some data");

// 使用ostringstream
std::ostringstream oss;

// 使用stringstream
std::stringstream ss;
```

## 实例

### 从字符串读取数据

下面是一个使用 `istringstream`​ 从字符串中读取整数和浮点数的例子：

## 实例

#include <iostream>  
#include <sstream>

int main() {  
    std::string data = "10 20.5";  
    std::istringstream iss(data);

    int i;  
    double d;

    iss >> i >> d;

    std::cout << "Integer: " << i << std::endl;  
    std::cout << "Double: " << d << std::endl;

    return 0;  
}

**输出结果：**

```
Integer: 10
Double: 20.5
```

### 向字符串写入数据

下面是一个使用 `ostringstream`​ 将数据写入字符串的例子：

## 实例

#include <iostream>  
#include <sstream>

int main() {  
    std::ostringstream oss;  
    int i = 100;  
    double d = 200.5;

    oss << i << " " << d;

    std::string result = oss.str();  
    std::cout << "Resulting string: " << result << std::endl;

    return 0;  
}

**输出结果：**

```
Resulting string: 100 200.5
```

### 使用stringstream进行读写操作

下面是一个使用 `stringstream`​ 同时进行读取和写入操作的例子：

## 实例

#include <iostream>  
#include <sstream>

int main() {  
    std::string data = "30 40.5";  
    std::stringstream ss(data);

    int i;  
    double d;

    // 从stringstream读取数据  
    ss >> i >> d;

    std::cout << "Read Integer: " << i << ", Double: " << d << std::endl;

    // 向stringstream写入数据  
    ss.str(""); // 清空stringstream  
    ss << "New data: " << 50 << " " << 60.7;

    std::string newData = ss.str();  
    std::cout << "New data string: " << newData << std::endl;

    return 0;  
}

**输出结果：**

```
Read Integer: 30, Double: 40.5
New data string: New data: 50 60.7
```

## 总结

​`sstream`​ 是 C++ 标准库中一个非常有用的组件，它简化了字符串和基本数据类型之间的转换。通过上述实例，我们可以看到如何使用 `istringstream`​、`ostringstream`​ 和 `stringstream`​ 来实现这些转换。掌握这些技能将帮助你在 C++ 编程中更加高效地处理字符串数据。
