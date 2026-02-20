# C++ 文件输入输出库 - <fstream>

在 C++ 中，` <fstream>`​ 是标准库中用于文件输入输出操作的类。它提供了一种方便的方式来读写文件。

​`fstream`​是`iostream`​库的一部分，支持文本和二进制文件的读写。

​`fstream`​类是`iostream`​库中的一个类，它继承自`istream`​和`ostream`​类，这意味着它既可以用于输入也可以用于输出。

### 语法

​`fstream`​ 的基本语法如下：

#include <fstream>

int main() {  
    std::fstream file; // 创建fstream对象  
    file.open("filename", mode); // 打开文件  
    // 进行文件操作  
    file.close(); // 关闭文件  
    return 0;  
}

其中`filename`​ 是文件的名称，`mode`​ 是打开文件的模式，常见的模式有：

* ​`std::ios::in`​：以输入模式打开文件。
* ​`std::ios::out`​：以输出模式打开文件。
* ​`std::ios::app`​：以追加模式打开文件。
* ​`std::ios::ate`​：打开文件并定位到文件末尾。
* ​`std::ios::trunc`​：打开文件并截断文件，即清空文件内容。

### 实例

写入文本文件:

## 实例

#include <fstream>  
#include <iostream>

int main() {  
    std::fstream file;  
    file.open("example.txt", std::ios::out); // 以输出模式打开文件

    if (!file) {  
        std::cerr << "Unable to open file!" << std::endl;  
        return 1; // 文件打开失败  
    }

    file << "Hello, World!" << std::endl; // 写入文本  
    file.close(); // 关闭文件

    return 0;  
}

在当前目录下创建一个名为`example.txt`​的文件，文件内容为：

```
Hello, World!
```

读取文本文件

## 实例

#include <fstream>  
#include <iostream>  
#include <string>

int main() {  
    std::fstream file;  
    file.open("example.txt", std::ios::in); // 以输入模式打开文件

    if (!file) {  
        std::cerr << "Unable to open file!" << std::endl;  
        return 1; // 文件打开失败  
    }

    std::string line;  
    while (getline(file, line)) { // 逐行读取  
        std::cout << line << std::endl;  
    }

    file.close(); // 关闭文件

    return 0;  
}

如果 `example.txt`​ 文件包含以下内容：

```
Hello, World!
This is a test file.
```

则程序将输出：

```
Hello, World!
This is a test file.
```

追加到文件:

## 实例

#include <fstream>  
#include <iostream>

int main() {  
    std::fstream file;  
    file.open("example.txt", std::ios::app); // 以追加模式打开文件

    if (!file) {  
        std::cerr << "Unable to open file!" << std::endl;  
        return 1; // 文件打开失败  
    }

    file << "Appending this line to the file." << std::endl; // 追加文本  
    file.close(); // 关闭文件

    return 0;  
}

​`example.txt`​ 文件原本包含以下内容：

```
Hello, World!
This is a test file.
```

执行上述程序后，文件内容将变为：

```
Hello, World!
This is a test file.
Appending this line to the file.
```
