# C++ 标准库 <numeric>

C++ 标准库中的 `<numeric>`​ 头文件提供了一组用于数值计算的函数模板，这些函数可以对容器中的元素进行各种数值操作，如求和、乘积、最小值、最大值等。这些函数模板非常强大，可以应用于任何类型的容器，包括数组、向量、列表等。

在使用 `<numeric>`​ 头文件中的函数之前，需要在你的 C++ 程序中包含这个头文件：

```
#include <numeric>
```

## 常用函数

### 1. `accumulate`​

​`accumulate`​ 函数用于计算容器中所有元素的总和。它接受三个参数：容器的开始迭代器、结束迭代器和初始值。

**语法**:

```
template <InputIterator Iter, class T>
T accumulate(Iter first, Iter last, T init);
```

**实例**:

## 实例

#include <iostream>  
#include <numeric>  
#include <vector>

int main() {  
    std::vector<int> v = {1, 2, 3, 4, 5};  
    int sum = std::accumulate(v.begin(), v.end(), 0);  
    std::cout << "Sum: " << sum << std::endl; // 输出: Sum: 15  
    return 0;  
}

### 2. `inner_product`​

​`inner_product`​ 函数用于计算两个容器中对应元素乘积的总和。

**语法**:

```
template <InputIterator1 Iter1, InputIterator2 Iter2, class T>
T inner_product(Iter1 first1, Iter1 last1, Iter2 first2, T init);
```

**实例**:

## 实例

#include <iostream>  
#include <numeric>  
#include <vector>

int main() {  
    std::vector<int> v1 = {1, 2, 3};  
    std::vector<int> v2 = {4, 5, 6};  
    int product\_sum = std::inner_product(v1.begin(), v1.end(), v2.begin(), 0);  
    std::cout << "Product Sum: " << product\_sum << std::endl; // 输出: Product Sum: 32  
    return 0;  
}

### 3. `partial_sum`​

​`partial_sum`​ 函数用于计算容器中元素的部分和，并将结果存储在另一个容器中。

**语法**:

```
template <InputIterator InIter, OutputIterator OutIter>
OutIter partial_sum(InIter first, InIter last, OutIter result);
```

**实例**:

## 实例

#include <iostream>  
#include <numeric>  
#include <vector>

int main() {  
    std::vector<int> v = {1, 2, 3, 4};  
    std::vector<int> result(v.size());  
    std::partial_sum(v.begin(), v.end(), result.begin());  
    for (int i : result) {  
        std::cout << i << " "; // 输出: 1 3 6 10  
    }  
    return 0;  
}

### 4. `adjacent_difference`​

​`adjacent_difference`​ 函数用于计算容器中相邻元素的差值，并将结果存储在另一个容器中。

**语法**:

```
template <InputIterator InIter, OutputIterator OutIter>
OutIter adjacent_difference(InIter first, InIter last, OutIter result);
```

**实例**:

## 实例

#include <iostream>  
#include <numeric>  
#include <vector>

int main() {  
    std::vector<int> v = {1, 2, 3, 4};  
    std::vector<int> result(v.size() - 1);  
    std::adjacent_difference(v.begin(), v.end(), result.begin());  
    for (int i : result) {  
        std::cout << i << " "; // 输出: 1 1 1  
    }  
    return 0;  
}

### 5. std::gcd

使用 std::gcd 计算两个整数的最大公约数：

## 实例

#include <iostream>  
#include <numeric>

int main() {  
    int a = 48;  
    int b = 18;  
    int result = std::gcd(a, b);  // 计算 48 和 18 的最大公约数  
    std::cout << "GCD: " << result << std::endl;  // 输出 6  
    return 0;  
}

### 6. std::lcm

使用 std::lcm 计算两个整数的最小公倍数：

## 实例

#include <iostream>  
#include <numeric>

int main() {  
    int a = 48;  
    int b = 18;  
    int result = std::lcm(a, b);  // 计算 48 和 18 的最小公倍数  
    std::cout << "LCM: " << result << std::endl;  // 输出 144  
    return 0;  
}

### 7. std::iota

## 实例

#include <iostream>  
#include <numeric>  // 包含 numeric 头文件  
#include <vector>

int main() {  
    std::vector<int> v(5); // 创建一个包含5个元素的向量

    // 使用 std::iota 填充向量，起始值为1  
    std::iota(std::begin(v), std::end(v), 1);

    // 输出填充后的向量  
    for (int i : v) {  
        std::cout << i << " ";  
    }  
    std::cout << std::endl; // 输出: 1 2 3 4 5

    return 0;  
}

使用 std::iota 填充范围内的序列值。

```
template<class ForwardIt, class T>
void iota(ForwardIt first, ForwardIt last, T value);
```

### 8.查找最大值与最小值

​`min_element`​ 和 `max_element`​ 函数用于找到容器中的最大值和最小值。

## 实例

#include <iostream>  
#include <numeric>  
#include <vector>  
#include <algorithm> // 为了使用 std::min_element 和 std::max_element

int main() {  
    // 定义一个包含整数的向量  
    std::vector<int> v = {3, 1, 4, 1, 5, 9};

    // 计算最小值和最大值  
    int min\_val = *std::min_element(v.begin(), v.end());  
    int max\_val = *std::max_element(v.begin(), v.end());

    // 计算总和  
    int sum\_val = std::accumulate(v.begin(), v.end(), 0);

    // 计算平均值  
    double avg\_val = static_cast<double>(sum\_val) / v.size();

    // 输出结果  
    std::cout << "Min: " << min\_val << std::endl;  
    std::cout << "Max: " << max\_val << std::endl;  
    std::cout << "Sum: " << sum\_val << std::endl;  
    std::cout << "Average: " << avg\_val << std::endl;

    return 0;  
}

输出结果为：

```
Min: 1
Max: 9
Sum: 23
Average: 3.83333
```
