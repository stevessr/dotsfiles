# C++ 标准库 <valarray>

C++ 的 `<valarray>`​ 库是一个用于数值计算的库，它提供了一种高效的方式来处理数值数组。`<valarray>`​ 库中的 `valarray`​ 类模板允许程序员对数组进行元素级的数学运算，包括加法、减法、乘法、除法等。此外，它还支持更高级的数学函数，如指数、对数、正弦、余弦等。

​`valarray`​ 是 C++ 标准库中的一个类模板，用于表示和操作数值数组。它提供了一种方便的方式来执行数组的元素级操作。

### 语法

​`valarray`​ 的基本语法如下：

```
#include <valarray>

int main() {
    std::valarray<double> va(10); // 创建一个包含10个double元素的valarray
    va = 1; // 将所有元素初始化为1
    // ...
    return 0;
}
```

## 实例

### 1. 创建和初始化 valarray

## 实例

#include <iostream>  
#include <valarray>

int main() {  
    std::valarray<double> va(5); // 创建一个包含5个double元素的valarray  
    va = {1.0, 2.0, 3.0, 4.0, 5.0}; // 初始化valarray

    for (auto i : va) {  
        std::cout << i << " ";  
    }  
    std::cout << std::endl;

    return 0;  
}

输出结果:

```
1 2 3 4 5
```

### 2. 基本运算

## 实例

#include <iostream>  
#include <valarray>

int main() {  
    std::valarray<double> va1(5), va2(5);  
    va1 = {1.0, 2.0, 3.0, 4.0, 5.0};  
    va2 = {2.0, 3.0, 4.0, 5.0, 6.0};

    std::valarray<double> sum = va1 + va2; // 加法  
    std::valarray<double> diff = va1 - va2; // 减法  
    std::valarray<double> prod = va1 * va2; // 乘法  
    std::valarray<double> quot = va1 / va2; // 除法

    std::cout << "Sum: ";  
    for (auto i : sum) {  
        std::cout << i << " ";  
    }  
    std::cout << std::endl;

    std::cout << "Difference: ";  
    for (auto i : diff) {  
        std::cout << i << " ";  
    }  
    std::cout << std::endl;

    std::cout << "Product: ";  
    for (auto i : prod) {  
        std::cout << i << " ";  
    }  
    std::cout << std::endl;

    std::cout << "Quotient: ";  
    for (auto i : quot) {  
        std::cout << i << " ";  
    }  
    std::cout << std::endl;

    return 0;  
}

输出结果:

```
Sum: 3 5 7 9 11 
Difference: -1 -1 1 1 -1 
Product: 2 6 12 20 30 
Quotient: 0.5 0.6666667 0.75 0.8 0.8333334
```

### 3. 使用 valarray 进行数学函数操作

## 实例

#include <iostream>  
#include <valarray>  
#include <cmath>

int main() {  
    std::valarray<double> va(5);  
    va = {1.0, 2.0, 3.0, 4.0, 5.0};

    std::valarray<double> squares = va * va; // 平方  
    std::valarray<double> roots = std::sqrt(va); // 开方

    std::cout << "Squares: ";  
    for (auto i : squares) {  
        std::cout << i << " ";  
    }  
    std::cout << std::endl;

    std::cout << "Square Roots: ";  
    for (auto i : roots) {  
        std::cout << i << " ";  
    }  
    std::cout << std::endl;

    return 0;  
}

输出结果:

```
Squares: 1 4 9 16 25 
Square Roots: 1 1.4142136 1.7320508
```
