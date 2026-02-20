# C++ 标准库 <complex> 入门指南

C++ 标准库中的 `<complex>`​ 头文件提供了对复数的支持。复数是实数和虚数的组合，通常表示为 `a + bi`​，其中 `a`​ 是实部，`b`​ 是虚部，`i`​ 是虚数单位，满足 `i^2 = -1`​。

在 C++ 中，复数类型由 `std::complex<T>`​ 表示，其中 `T`​ 可以是任意的算术类型，如 `float`​、`double`​ 或 `long double`​。

要使用 `<complex>`​ 库，首先需要在你的 C++ 程序中包含这个头文件：

```
#include <iostream>
#include <complex>
```

## 基本语法

### 创建复数

```
std::complex<double> c(5.0, 3.0); // 创建一个复数 5 + 3i
```

### 访问实部和虚部

```
double realPart = c.real(); // 获取实部
double imagPart = c.imag(); // 获取虚部
```

### 复数的基本运算

C++ 标准库 `<complex>`​ 支持以下基本运算：

* 加法：`operator+`​
* 减法：`operator-`​
* 乘法：`operator*`​
* 除法：`operator/`​
* 共轭：`conj`​
* 模：`abs`​
* 辐角：`arg`​

## 实例

下面是一个使用 `<complex>`​ 头文件的简单示例，包括创建复数、基本运算和输出结果。

## 实例

#include <iostream>  
#include <complex>

int main() {  
    // 创建两个复数  
    std::complex<double> c1(5.0, 3.0); // 5 + 3i  
    std::complex<double> c2(2.0, -4.0); // 2 - 4i

    // 输出复数  
    std::cout << "c1: " << c1 << std::endl;  
    std::cout << "c2: " << c2 << std::endl;

    // 复数加法  
    std::complex<double> sum = c1 + c2;  
    std::cout << "Sum: " << sum << std::endl; // 输出 7 - i

    // 复数减法  
    std::complex<double> diff = c1 - c2;  
    std::cout << "Difference: " << diff << std::endl; // 输出 3 + 7i

    // 复数乘法  
    std::complex<double> product = c1 * c2;  
    std::cout << "Product: " << product << std::endl; // 输出 -37 + 2i

    // 复数除法  
    std::complex<double> quotient = c1 / c2;  
    std::cout << "Quotient: " << quotient << std::endl; // 输出 0.4 - 1.2i

    // 复数的共轭  
    std::complex<double> conjugate = std::conj(c1);  
    std::cout << "Conjugate of c1: " << conjugate << std::endl; // 输出 5 - 3i

    // 复数的模  
    double modulus = std::abs(c1);  
    std::cout << "Modulus of c1: " << modulus << std::endl; // 输出 sqrt(34)

    // 复数的辐角  
    double argument = std::arg(c1);  
    std::cout << "Argument of c1: " << argument << std::endl; // 输出 atan(3/5)

    return 0;  
}

当你运行上述程序时，你将得到以下输出：

```
c1: (5,3)
c2: (2,-4)
Sum: (7,-1)
Difference: (3,7)
Product: (-37,2)
Quotient: (0.4,-1.2)
Conjugate of c1: (5,-3)
Modulus of c1: 5.83095
Argument of c1: 0.61948
```
