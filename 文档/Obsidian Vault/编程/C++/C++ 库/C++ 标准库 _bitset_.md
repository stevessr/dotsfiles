# C++ 标准库 <bitset>

在 C++ 编程中，`<bitset>`​ 是标准库的一部分，它提供了一种方式来操作固定大小的位集合。

位集合是一个由位（bit）组成的数组，每个位可以是 0 或 1。

​`<bitset>`​ 提供了一种高效的方式来存储和操作二进制数据，特别适合需要位级操作的场景，如标志位管理、位掩码操作等。

​`bitset`​ 是一个模板类，其模板参数定义了位集合的大小。例如，`bitset<32>`​ 表示一个包含 32 位的位集合。

### 语法

以下是使用 `bitset`​ 的基本语法：

```
#include <bitset>

// 声明一个大小为N的bitset
std::bitset<N> b;

// 初始化bitset
b = std::bitset<N>(value);

// 访问位集合中的单个位
bool bit = b[i];
```

### std::bitset 的基本用法

std::bitset 是一个模板类，用于表示固定大小的二进制位序列。它的模板参数是位数（N），表示二进制序列的长度。

定义 std::bitset：

```
std::bitset<8> bits;  // 定义一个 8 位的二进制序列
```

初始化 std::bitset：

* 默认初始化：所有位为 `0`​。
* 从整数初始化：将整数转换为二进制。
* 从字符串初始化：将字符串解析为二进制。

## 实例

std::bitset<8> bits1;               // 默认初始化：00000000  
std::bitset<8> bits2(42);           // 从整数初始化：00101010  
std::bitset<8> bits3("10101010");   // 从字符串初始化：10101010

### 常用成员函数

std::bitset 提供了丰富的成员函数来操作二进制位。

访问和修改位:

* ​`operator[]`​：访问或修改某一位。
* ​`set()`​：将某一位或所有位设置为 `1`​。
* ​`reset()`​：将某一位或所有位设置为 `0`​。
* ​`flip()`​：翻转某一位或所有位。

## 实例

std::bitset<8> bits("00001111");  
bits[0] = 1;          // 修改第 0 位：00001111 -> 00001111  
bits.set(4);          // 设置第 4 位：00001111 -> 00011111  
bits.reset(1);        // 重置第 1 位：00011111 -> 00011101  
bits.flip();          // 翻转所有位：00011101 -> 11100010

查询位信息:

* ​`count()`​：返回 `1`​ 的个数。
* ​`size()`​：返回位数。
* ​`test(pos)`​：检查某一位是否为 `1`​。
* ​`all()`​：检查是否所有位都为 `1`​。
* ​`any()`​：检查是否有任何一位为 `1`​。
* ​`none()`​：检查是否所有位都为 `0`​。

## 实例

std::bitset<8> bits("10101010");  
std::cout << "Count of 1s: " << bits.count() << std::endl;  // 输出 4  
std::cout << "Size: " << bits.size() << std::endl;          // 输出 8  
std::cout << "Is bit 3 set? " << bits.test(3) << std::endl; // 输出 1 (true)  
std::cout << "All bits set? " << bits.all() << std::endl;   // 输出 0 (false)

转换为其他类型:

* ​`to_ulong()`​：将 `std::bitset`​ 转换为 `unsigned long`​。
* ​`to_ullong()`​：将 `std::bitset`​ 转换为 `unsigned long long`​。
* ​`to_string()`​：将 `std::bitset`​ 转换为字符串。

## 实例

std::bitset<8> bits("10101010");  
unsigned long num = bits.to_ulong();  // 转换为整数：170  
std::string str = bits.to_string();   // 转换为字符串："10101010"

### 位操作

std::bitset 支持常见的位操作，如按位与、按位或、按位异或和按位取反。

* ​`&`​：按位与
* ​`|`​：按位或
* ​`^`​：按位异或
* ​`~`​：按位取反

## 实例

std::bitset<8> bits1("10101010");  
std::bitset<8> bits2("11110000");

std::bitset<8> result\_and = bits1 & bits2;  // 按位与：10100000  
std::bitset<8> result\_or = bits1 | bits2;   // 按位或：11111010  
std::bitset<8> result\_xor = bits1 ^ bits2;  // 按位异或：01011010  
std::bitset<8> result\_not = \~bits1;         // 按位取反：01010101

## 实例

基本使用:

## 实例

#include <iostream>  
#include <bitset>

int main() {  
    std::bitset<8> b("11001010"); // 从字符串初始化  
    std::cout << "Initial bitset: " << b << std::endl;

    // 访问特定位置的位  
    std::cout << "Bit at position 3: " << b[3] << std::endl;

    // 修改位  
    b[3] = 1;  
    std::cout << "Modified bitset: " << b << std::endl;

    // 翻转位  
    b.flip();  
    std::cout << "Flipped bitset: " << b << std::endl;

    return 0;  
}

输出结果：

```
Initial bitset: 11001010
Bit at position 3: 0
Modified bitset: 11001011
Flipped bitset: 00110110
```

位操作:

## 实例

#include <iostream>  
#include <bitset>

int main() {  
    std::bitset<8> b1("10101010");  
    std::bitset<8> b2("11110000");

    // 位与操作  
    std::bitset<8> b\_and = b1 & b2;  
    std::cout << "Bitwise AND: " << b\_and << std::endl;

    // 位或操作  
    std::bitset<8> b\_or = b1 | b2;  
    std::cout << "Bitwise OR: " << b\_or << std::endl;

    // 位异或操作  
    std::bitset<8> b\_xor = b1 ^ b2;  
    std::cout << "Bitwise XOR: " << b\_xor << std::endl;

    // 位非操作  
    std::bitset<8> b\_not = \~b1;  
    std::cout << "Bitwise NOT: " << b\_not << std::endl;

    return 0;  
}

输出结果：

```
Bitwise AND: 10100000
Bitwise OR: 11111010
Bitwise XOR: 01111010
Bitwise NOT: 01010101
```

循环遍历位集合:

## 实例

#include <iostream>  
#include <bitset>

int main() {  
    std::bitset<8> b("10101010");

    // 循环遍历bitset中的位  
    for (size_t i = 0; i < b.size(); ++i) {  
        std::cout << b[i];  
    }  
    std::cout << std::endl;

    return 0;  
}

输出结果：

```
10101010
```

​`bitset`​ 是C++标准库中一个非常有用的工具，它允许程序员以一种直观和高效的方式处理位级数据。通过上述实例，我们可以看到如何声明、初始化、访问、修改以及进行位操作。这些功能在处理二进制数据或需要进行位级控制的场合非常有用。

### 注意事项

* ​`std::bitset`​ 的大小是固定的，在编译时确定。
* 如果位数超过 `unsigned long`​ 或 `unsigned long long`​ 的位数，`to_ulong()`​ 和 `to_ullong()`​ 会抛出 `std::overflow_error`​ 异常。
* ​`std::bitset`​ 不支持动态调整大小，如果需要动态位集，可以考虑 `std::vector<bool>`​。
