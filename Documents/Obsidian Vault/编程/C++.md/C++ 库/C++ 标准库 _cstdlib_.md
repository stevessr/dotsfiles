# C++ 标准库 <cstdlib>

​`<cstdlib>`​ 是 C++ 标准库中的一个头文件，提供了各种通用工具函数，包括内存分配、进程控制、环境查询、排序和搜索、数学转换、伪随机数生成等。这些函数最初来自 C 标准库 `<stdlib.h>`​，在 C++ 中进行了标准化和扩展。

### 语法

在 C++ 程序中，要使用 `cstdlib`​ 中的函数，需要先包含这个头文件：

```
#include <cstdlib>
```

### 常用函数

​`cstdlib`​ 中包含了许多有用的函数，以下是一些常用的函数及其简要说明：

1. ​`exit(int status)`​: 终止程序执行，并返回一个状态码。
2. ​`system(const char* command)`​: 执行一个命令行字符串。
3. ​`malloc(size_t size)`​: 分配指定大小的内存。
4. ​`free(void* ptr)`​: 释放之前分配的内存。
5. ​`atoi(const char* str)`​: 将字符串转换为整数。
6. ​`atof(const char* str)`​: 将字符串转换为浮点数。
7. ​`rand()`​: 生成一个随机数。
8. ​`srand(unsigned int seed)`​: 设置随机数生成器的种子。

### 实例

以下是一些使用 `cstdlib`​ 中函数的实例：

### 实例 1：使用 `exit`​ 函数

## 实例

#include <iostream>  
#include <cstdlib>

int main() {  
    std::cout << "This program will exit now." << std::endl;  
    exit(0); // 正常退出程序  
    return 0; // 这行代码不会被执行  
}

**输出结果：**

```
This program will exit now.
```

### 实例 2：使用 `system`​ 函数

## 实例

#include <iostream>  
#include <cstdlib>

int main() {  
    std::cout << "Executing a system command: dir" << std::endl;  
    system("dir"); // 在 Windows 上显示当前目录的文件和文件夹  
    return 0;  
}

**输出结果：**

```
Executing a system command: dir
```

然后显示当前目录的文件和文件夹列表。

### 实例 3：使用 `malloc`​ 和 `free`​ 函数

## 实例

#include <iostream>  
#include <cstdlib>

int main() {  
    int* ptr = (int*)malloc(10 * sizeof(int)); // 分配内存  
    if (ptr == NULL) {  
        std::cout << "Memory allocation failed." << std::endl;  
        return 1;  
    }

    for (int i = 0; i < 10; ++i) {  
        ptr[i] = i * i; // 使用分配的内存  
    }

    for (int i = 0; i < 10; ++i) {  
        std::cout << "Element " << i << ": " << ptr[i] << std::endl;  
    }

    free(ptr); // 释放内存  
    return 0;  
}

**输出结果：**

```
Element 0: 0
Element 1: 1
Element 2: 4
...
Element 9: 81
```

### 实例 4：使用 `atoi`​ 和 `atof`​ 函数

## 实例

#include <iostream>  
#include <cstdlib>

int main() {  
    std::string str1 = "123";  
    std::string str2 = "456.78";

    int num1 = std::atoi(str1.c_str()); // 将字符串转换为整数  
    double num2 = std::atof(str2.c_str()); // 将字符串转换为浮点数

    std::cout << "Integer: " << num1 << std::endl;  
    std::cout << "Float: " << num2 << std::endl;  
    return 0;  
}

**输出结果：**

```
Integer: 123
Float: 456.78
```

### 实例 5：使用 `rand`​ 和 `srand`​ 函数

## 实例

#include <cstdlib>  
#include <iostream>  
#include <ctime>

int main() {  
    std::srand(std::time(nullptr));  // 使用当前时间作为随机数种子  
    for (int i = 0; i < 5; ++i) {  
        std::cout << std::rand() % 100 << " ";  // 生成0到99之间的随机数  
    }  
    std::cout << std::endl;

    return 0;  
}

**输出结果：**

```
53 22 62 68 62 
```
