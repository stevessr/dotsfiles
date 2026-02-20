# C++ 标准库 <iomanip>

​`<iomanip>`​ 是 C++ 标准库中的一个头文件，它提供了对输入/输出流的格式化操作。

​`iomanip`​ 库中的函数允许开发者控制输出格式，如设置小数点后的位数、设置宽度、对齐方式等。

​`iomanip`​ 是 Input/Output Manipulators 的缩写，它提供了一组操作符，用于控制 C++ 标准库中的输入/输出流的格式，适用以下场景：

* 科学计算中浮点数格式的处理；
* 数据对齐与美化；
* 显示特定进制或格式的数值。

### 语法

​`iomanip`​ 库中的函数通常与 `<<`​ 和 `>>`​ 操作符一起使用，以实现对输出流的控制。

以下是一些常用的 `iomanip`​ 函数：

|**函数/操纵符**|**功能**|**实例代码**|**输出结果**|
| ------| ------------------------------------| ------| ------|
|​`std::setw(int n)`​|设置字段宽度，为下一次输出指定宽度|​`std::cout << std::setw(5) << 42;`​|​`42`​|
|​`std::setfill(char)`​|设置填充字符（默认是空格）|​`std::cout << std::setfill('*') << std::setw(5) << 42;`​|​`***42`​|
|​`std::left`​|设置左对齐|​`std::cout << std::left << std::setw(5) << 42;`​|​`42`​|
|​`std::right`​|设置右对齐|​`std::cout << std::right << std::setw(5) << 42;`​|​`42`​|
|​`std::internal`​|符号靠左，其余靠右|​`std::cout << std::internal << std::setw(5) << -42;`​|​`- 42`​|
|​`std::setprecision(int)`​|设置浮点数的有效位数|​`std::cout << std::setprecision(3) << 3.14159;`​|​`3.14`​|
|​`std::fixed`​|设置定点格式输出浮点数|​`std::cout << std::fixed << std::setprecision(2) << 3.14159;`​|​`3.14`​|
|​`std::scientific`​|设置科学计数法格式输出浮点数|​`std::cout << std::scientific << 3.14159;`​|​`3.141590e+00`​|
|​`std::hex`​|设置整数以 16 进制显示|​`std::cout << std::hex << 42;`​|​`2a`​|
|​`std::oct`​|设置整数以 8 进制显示|​`std::cout << std::oct << 42;`​|​`52`​|
|​`std::dec`​|设置整数以 10 进制显示（默认）|​`std::cout << std::dec << 42;`​|​`42`​|
|​`std::showbase`​|显示进制前缀（如 `0x`​ 表示 16 进制）|​`std::cout << std::showbase << std::hex << 42;`​|​`0x2a`​|
|​`std::noshowbase`​|隐藏进制前缀（默认）|​`std::cout << std::noshowbase << std::hex << 42;`​|​`2a`​|
|​`std::uppercase`​|16 进制字母显示为大写|​`std::cout << std::uppercase << std::hex << 42;`​|​`2A`​|
|​`std::nouppercase`​|16 进制字母显示为小写（默认）|​`std::cout << std::nouppercase << std::hex << 42;`​|​`2a`​|
|​`std::showpos`​|在正数前显示 `+`​ 符号|​`std::cout << std::showpos << 42;`​|​`+42`​|
|​`std::noshowpos`​|不显示正数的 `+`​ 符号（默认）|​`std::cout << std::noshowpos << 42;`​|​`42`​|
|​`std::boolalpha`​|布尔值以 `true/false`​ 输出|​`std::cout << std::boolalpha << true;`​|​`true`​|
|​`std::noboolalpha`​|布尔值以 `1/0`​ 输出（默认）|​`std::cout << std::noboolalpha << true;`​|​`1`​|
|​`std::setbase(int n)`​|设置整数的进制（支持 8、10、16）|​`std::cout << std::setbase(16) << 42;`​|​`2a`​|
|​`std::resetiosflags`​|重置指定的流状态|​`std::cout << std::resetiosflags(std::ios::showbase) << std::hex << 42;`​|​`2a`​|
|​`std::setiosflags`​|设置指定的流状态|​`std::cout << std::setiosflags(std::ios::showbase) << std::hex << 42;`​|​`0x2a`​|

显示详细信息

## 实例

### 1. 设置宽度

使用 `setw`​ 可以设置输出的宽度。如果输出内容的字符数少于设置的宽度，剩余部分将用空格填充。

## 实例

#include <iostream>  
#include <iomanip>

int main() {  
    std::cout << std::setw(10) << "Hello" << std::endl;  
    return 0;  
}

**输出结果:**

```
Hello
```

### 2. 设置精度

使用 `setprecision`​ 可以设置设置浮点数的有效位数。

## 实例

#include <iostream>  
#include <iomanip>

int main() {  
    double pi = 3.141592653589793;  
    std::cout << "Default: " << pi << "\n";  
    std::cout << "Set precision (3): " << std::setprecision(3) << pi << "\n";  
    std::cout << "Set precision (7): " << std::setprecision(7) << pi << "\n";  
    return 0;  
}

**输出结果:**

```
Default: 3.14159
Set precision (3): 3.14
Set precision (7): 3.141593
```

### 3. 固定小数点和科学计数法

​`fixed`​ 和 `scientific`​ 可以控制浮点数的输出格式。

## 实例

#include <iostream>  
#include <iomanip>

int main() {  
    double num = 123456789.0;  
    std::cout << "Fixed: " << std::fixed << num << std::endl;  
    std::cout << "Scientific: " << std::scientific << num << std::endl;  
    return 0;  
}

**输出结果:**

```
Fixed: 123456789.000000
Scientific: 1.23456789e+08
```

### 4. 设置填充字符

使用 `setfill`​ 可以设置填充字符，通常与 `setw`​ 一起使用。

## 实例

#include <iostream>  
#include <iomanip>

int main() {  
    std::cout << std::setfill('*') << std::setw(10) << "World" << std::endl;  
    return 0;  
}

**输出结果:**

```
*****World
```

### 5. 设置和重置格式标志

​`setiosflags`​ 和 `resetiosflags`​ 可以设置或重置流的格式标志。

## 实例

#include <iostream>  
#include <iomanip>

int main() {  
    std::cout << std::setiosflags(std::ios::uppercase) << std::hex << 255 << std::endl;  
    std::cout << std::resetiosflags(std::ios::uppercase) << std::hex << 255 << std::endl;  
    return 0;  
}

**输出结果:**

```
FF
ff
```
