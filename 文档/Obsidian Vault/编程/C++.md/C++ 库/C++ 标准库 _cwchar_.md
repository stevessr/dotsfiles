# C++ 标准库 <cwchar>

​`<cwchar>`​ 是 C++ 标准库中的一个头文件，提供了处理宽字符（`wchar_t`​）和宽字符串的函数。这些函数大部分来自 C 标准库的 `<wchar.h>`​，用于处理宽字符的输入输出、内存操作、字符串操作和其他与宽字符相关的功能。

## 语法

​`cwchar`​ 头文件中定义的函数通常具有与标准字符处理函数相似的名称，但以 `w`​ 开头，例如 `wprintf`​、`wscanf`​ 等。这些函数的参数和返回类型也与相应的标准字符函数不同，它们使用宽字符类型 `wchar_t`​。

### 基本类型

* ​`wchar_t`​：宽字符类型，用于存储宽字符。
* ​`wint_t`​：用于存储宽字符函数的返回值。

## 常用函数

### 1. **宽字符输入输出**

* ​`fgetwc`​：从文件流中读取宽字符。
* ​`fputwc`​：向文件流中写入宽字符。
* ​`fgetws`​：从文件流中读取宽字符串。
* ​`fputws`​：向文件流中写入宽字符串。

## 实例

#include <cwchar>  
#include <iostream>

int main() {  
    // 使用 fputws 和 fgetws 进行宽字符串输入输出  
    const wchar_t *filename = L"example.txt";  
    FILE *file = std::fopen("example.txt", "w");  
    if (file) {  
        std::fputws(L"Hello, 世界!\n", file);  
        std::fclose(file);  
    }

    file = std::fopen("example.txt", "r");  
    if (file) {  
        wchar_t buffer[256];  
        if (std::fgetws(buffer, 256, file)) {  
            std::wcout << L"Read from file: " << buffer;  
        }  
        std::fclose(file);  
    }

    return 0;  
}

输出结果为：

```
Read from file: Hello, 世界!
```

### 2. **宽字符和宽字符串操作**

* ​`wcscpy`​：拷贝宽字符串。
* ​`wcslen`​：获取宽字符串长度。
* ​`wcscmp`​：比较宽字符串。
* ​`wcsncpy`​：拷贝指定长度的宽字符串。

## 实例

#include <cwchar>  
#include <iostream>

int main() {  
    wchar_t str1[100] = L"Hello";  
    wchar_t str2[100] = L"World";

    // 宽字符串拷贝  
    std::wcscpy(str1, L"Hello, 世界!");  
    std::wcout << L"Copied string: " << str1 << std::endl;

    // 宽字符串长度  
    size_t len = std::wcslen(str1);  
    std::wcout << L"Length of string: " << len << std::endl;

    // 宽字符串比较  
    int result = std::wcscmp(str1, str2);  
    std::wcout << L"Comparison result: " << result << std::endl;

    // 宽字符串部分拷贝  
    std::wcsncpy(str2, str1, 5);  
    str2[5] = L'\0';  // 确保以空字符结束  
    std::wcout << L"Partially copied string: " << str2 << std::endl;

    return 0;  
}

输出结果为：

```
Copied string: Hello, 
```

### 3. **宽字符分类和转换**

* ​`iswalpha`​：判断宽字符是否为字母。
* ​`iswdigit`​：判断宽字符是否为数字。
* ​`towlower`​：将宽字符转换为小写。
* ​`towupper`​：将宽字符转换为大写。

## 实例

#include <cwchar>  
#include <iostream>

int main() {  
    wchar_t ch = L'A';

    // 判断宽字符是否为字母  
    if (std::iswalpha(ch)) {  
        std::wcout << ch << L" is an alphabetic character." << std::endl;  
    }

    // 判断宽字符是否为数字  
    ch = L'9';  
    if (std::iswdigit(ch)) {  
        std::wcout << ch << L" is a digit." << std::endl;  
    }

    // 宽字符转换为小写  
    ch = L'G';  
    wchar_t lower\_ch = std::towlower(ch);  
    std::wcout << L"Lowercase of " << ch << L" is " << lower\_ch << std::endl;

    // 宽字符转换为大写  
    ch = L'a';  
    wchar_t upper\_ch = std::towupper(ch);  
    std::wcout << L"Uppercase of " << ch << L" is " << upper\_ch << std::endl;

    return 0;  
}

输出结果为：

```
A is an alphabetic character.
9 is a digit.
Lowercase of G is g
Uppercase of a is A
```

### 4. **宽字符和宽字符串的输入输出**

* ​`wprintf`​：宽字符格式化输出。
* ​`wscanf`​：宽字符格式化输入。
* ​`swprintf`​：将格式化宽字符写入宽字符串。
* ​`swscanf`​：从宽字符串中读取格式化宽字符。

## 实例

#include <cwchar>  
#include <iostream>

int main() {  
    wchar_t buffer[100];

    // 宽字符格式化输出  
    std::wprintf(L"Formatted output: %d %s\n", 42, L"Hello");

    // 宽字符格式化输入  
    std::wprintf(L"Enter a number and a string: ");  
    std::wscanf(L"%d %ls", &buffer);  
    std::wprintf(L"You entered: %ls\n", buffer);

    // 将格式化宽字符写入宽字符串  
    std::swprintf(buffer, 100, L"Formatted %d %s", 42, L"output");  
    std::wcout << L"Buffer: " << buffer << std::endl;

    // 从宽字符串中读取格式化宽字符  
    int number;  
    wchar_t word[100];  
    std::swscanf(buffer, L"Formatted %d %s", &number, word);  
    std::wcout << L"Parsed number: " << number << L", word: " << word << std::endl;

    return 0;  
}

输出结果为：

```
Formatted output: 42 H
Enter a number and a string: runoob
You entered: 
Buffer: Formatted 42 o
Parsed number: 42, word: o
```
