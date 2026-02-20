# C++ 标准库 <type_traits>

​`<type_traits>`​ 是 C++ 标准库中一个非常有用的头文件，它包含了一组编译时检查类型特性的工具。这些工具可以帮助开发者在编译时确定类型的特性，从而实现更安全、更灵活的代码。

​`<type_traits>`​ 头文件定义了一组模板，这些模板可以用于查询和操作类型属性。这些属性包括但不限于：

* 是否是整数类型
* 是否是浮点类型
* 是否是指针类型
* 是否是引用类型
* 是否是可调用的（函数或函数指针）

## 语法

​`<type_traits>`​ 中的模板通常使用 `std::`​ 前缀，例如 `std::is_integral<T>::value`​ 用于检查类型 `T`​ 是否是整数类型。这里的 `value`​ 是一个静态常量，其值为 `true`​ 或 `false`​。

以下是一些常用的 **type_traits** 功能：

**基本类型判断**：

* ​`std::is_void<T>`​: 判断类型 `T`​ 是否为 `void`​。
* ​`std::is_integral<T>`​: 判断类型 `T`​ 是否为整型。
* ​`std::is_floating_point<T>`​: 判断类型 `T`​ 是否为浮点型。
* ​`std::is_array<T>`​: 判断类型 `T`​ 是否为数组类型。
* ​`std::is_pointer<T>`​: 判断类型 `T`​ 是否为指针类型。
* ​`std::is_reference<T>`​: 判断类型 `T`​ 是否为引用类型。
* ​`std::is_const<T>`​: 判断类型 `T`​ 是否为 `const`​ 修饰。

**类型修饰**：

* ​`std::remove_const<T>`​: 移除类型 `T`​ 的 `const`​ 修饰。
* ​`std::remove_volatile<T>`​: 移除类型 `T`​ 的 `volatile`​ 修饰。
* ​`std::remove_cv<T>`​: 同时移除类型 `T`​ 的 `const`​ 和 `volatile`​ 修饰。
* ​`std::remove_reference<T>`​: 移除类型 `T`​ 的引用修饰。
* ​`std::remove_pointer<T>`​: 移除类型 `T`​ 的指针修饰。

**类型转换**：

* ​`std::add_const<T>`​: 为类型 `T`​ 添加 `const`​ 修饰。
* ​`std::add_volatile<T>`​: 为类型 `T`​ 添加 `volatile`​ 修饰。
* ​`std::add_cv<T>`​: 同时为类型 `T`​ 添加 `const`​ 和 `volatile`​ 修饰。
* ​`std::add_pointer<T>`​: 为类型 `T`​ 添加指针修饰。
* ​`std::add_lvalue_reference<T>`​: 为类型 `T`​ 添加左值引用修饰。
* ​`std::add_rvalue_reference<T>`​: 为类型 `T`​ 添加右值引用修饰。

**类型特性检测**：

* ​`std::is_same<T, U>`​: 判断类型 `T`​ 和 `U`​ 是否相同。
* ​`std::is_base_of<Base, Derived>`​: 判断类型 `Base`​ 是否为类型 `Derived`​ 的基类。
* ​`std::is_convertible<From, To>`​: 判断类型 `From`​ 是否能转换为类型 `To`​。

**条件类型**：

* ​`std::conditional<Condition, T, F>`​: 如果 `Condition`​ 为 `true`​，则类型为 `T`​，否则为 `F`​。
* ​`std::enable_if<Condition, T>`​: 如果 `Condition`​ 为 `true`​，则类型为 `T`​，否则此模板不参与重载决议。

## 实例

以下是一些使用 `<type_traits>`​ 的实例，以及它们的输出结果。

### 检查是否是整数类型

## 实例

#include <iostream>  
#include <type_traits>

int main() {  
    std::cout << "int is integral: " << std::is_integral<int>::value << std::endl;  
    std::cout << "float is integral: " << std::is_integral<float>::value << std::endl;  
    std::cout << "char is integral: " << std::is_integral<char>::value << std::endl;  
    return 0;  
}

**输出结果:**

```
int is integral: 1
float is integral: 0
char is integral: 1
```

### 检查是否是浮点类型

## 实例

#include <iostream>  
#include <type_traits>

int main() {  
    std::cout << "int is floating_point: " << std::is_floating_point<int>::value << std::endl;  
    std::cout << "float is floating_point: " << std::is_floating_point<float>::value << std::endl;  
    std::cout << "double is floating_point: " << std::is_floating_point<double>::value << std::endl;  
    return 0;  
}

输出结果:

```
int is floating_point: 0
float is floating_point: 1
double is floating_point: 1
```

### 检查是否是指针类型

## 实例

#include <iostream>  
#include <type_traits>

int main() {  
    int a = 10;  
    int* p = &a;  
    std::cout << "int* is a pointer: " << std::is_pointer<int*>::value << std::endl;  
    std::cout << "int is a pointer: " << std::is_pointer<int>::value << std::endl;  
    return 0;  
}

输出结果:

```
int* is a pointer: 1
int is a pointer: 0
```

### 检查是否是引用类型

## 实例

#include <iostream>  
#include <type_traits>

int main() {  
    int a = 10;  
    int& ref = a;  
    std::cout << "int& is a reference: " << std::is_reference<int&>::value << std::endl;  
    std::cout << "int is a reference: " << std::is_reference<int>::value << std::endl;  
    return 0;  
}

输出结果:

```
int& is a reference: 1
int is a reference: 0
```

### 检查是否是可调用的

## 实例

#include <iostream>  
#include <type_traits>  
#include <functional>

void func() {}

int main() {  
    std::cout << "int is callable: " << std::is_callable<int>::value << std::endl;  
    std::cout << "void() is callable: " << std::is_callable<void()>::value << std::endl;  
    std::cout << "func is callable: " << std::is_callable<decltype(func)>::value << std::endl;  
    return 0;  
}

输出结果:

```
int is callable: 0
void() is callable: 1
func is callable: 1
```

​`<type_traits>`​ 是 C++ 中一个非常有用的工具，它允许开发者在编译时检查和操作类型属性。这不仅可以提高代码的安全性，还可以使代码更加灵活和可重用。
