# C++ 标准输入输出 -- <iostream>

​`<iostream>`​库是 C++ 标准库中用于输入输出操作的头文件。

\<iostream\> 定义了几个常用的流类和操作符，允许程序与标准输入输出设备（如键盘和屏幕）进行交互。

以下是`<iostream>`​库的详细使用说明，包括其主要类和常见用法示例。

### 主要类

* ​`std::istream`​：用于输入操作的抽象基类。
* ​`std::ostream`​：用于输出操作的抽象基类。
* ​`std::iostream`​：继承自`std::istream`​和`std::ostream`​，用于同时进行输入和输出操作。
* ​`std::cin`​：标准输入流对象，通常与键盘关联。
* ​`std::cout`​：标准输出流对象，通常与屏幕关联。
* ​`std::cerr`​：标准错误输出流对象，不带缓冲，通常与屏幕关联。
* ​`std::clog`​：标准日志流对象，带缓冲，通常与屏幕关联。

### 常用操作符

* ​`>>`​：输入操作符，从输入流读取数据。
* ​`<<`​：输出操作符，将数据写入输出流。

### 基本用法

标准输入和输出:

## 实例

#include <iostream>

int main() {  
    int age;  
    std::string name;

    // 使用 std::cout 输出到屏幕  
    std::cout << "Enter your name: ";  
    // 使用 std::cin 从键盘读取输入  
    std::cin >> name;

    std::cout << "Enter your age: ";  
    std::cin >> age;

    // 输出读取到的数据  
    std::cout << "Hello, " << name << "! You are " << age << " years old." << std::endl;

    return 0;  
}

标准错误输出:

## 实例

#include <iostream>

int main() {  
    std::cerr << "An error occurred!" << std::endl;  
    return 0;  
}

标准日志输出:

## 实例

#include <iostream>

int main() {  
    std::clog << "This is a log message." << std::endl;  
    return 0;  
}

格式化输出:

使用`<iomanip>`​库可以对输出进行格式化，例如设置宽度、精度和对齐方式。

## 实例

#include <iostream>  
#include <iomanip>

int main() {  
    double pi = 3.14159;

    // 设置输出精度  
    std::cout << std::setprecision(3) << pi << std::endl;

    // 设置输出宽度和对齐方式  
    std::cout << std::setw(10) << std::left << pi << std::endl;  
    std::cout << std::setw(10) << std::right << pi << std::endl;

    return 0;  
}

### 流的状态检查:

可以检查输入输出流的状态，以确定操作是否成功。

## 实例

#include <iostream>

int main() {  
    int num;  
    std::cout << "Enter a number: ";  
    std::cin >> num;

    // 检查输入操作是否成功  
    if (std::cin.fail()) {  
        std::cerr << "Invalid input!" << std::endl;  
    } else {  
        std::cout << "You entered: " << num << std::endl;  
    }

    return 0;  
}

### 处理字符串输入

使用`std::getline`​函数可以读取包含空格的整行输入。

## 实例

#include <iostream>  
#include <string>

int main() {  
    std::string fullName;  
    std::cout << "Enter your full name: ";  
    std::getline(std::cin, fullName);  
    std::cout << "Hello, " << fullName << "!" << std::endl;

    return 0;  
}

以上示例展示了`<iostream>`​库的基本用法和常见操作，帮助你在C++程序中进行输入输出处理。
