# C++ 标准库 <ctime>

C++ 标准库提供了丰富的功能，其中 `<ctime>`​ 是处理时间和日期的标准库之一。它提供了一组函数，用于获取当前时间、日期以及执行时间相关的计算。

​`<ctime>`​ 库定义了一组与时间相关的函数和类型，这些函数和类型允许程序员在程序中处理时间。它包括：

* ​`time_t`​：表示时间的类型，通常是一个长整型。
* ​`tm`​：一个结构体，用于表示时间的各个部分，如年、月、日、小时等。
* 一系列函数，如 `time()`​, `localtime()`​, `gmtime()`​, `strftime()`​ 等。

## 语法

以下是 `<ctime>`​ 库中一些常用函数的基本语法：

* 获取当前时间（以秒为单位，从1970年1月1日开始计算）：

  ```
  time_t t = time(NULL);
  ```
* 将 `time_t`​ 类型的时间转换为 `tm`​ 结构体：

  ```
  struct tm *tm = localtime(&t);
  ```
* 将 `time_t`​ 类型的时间转换为协调世界时（UTC）的 `tm`​ 结构体：

  ```
  struct tm *tm_utc = gmtime(&t);
  ```
* 格式化时间：

  ```
  char buffer[80];
  strftime(buffer, 80, "%Y-%m-%d %H:%M:%S", tm);
  ```

## 实例

下面是一个使用 `<ctime>`​ 库的简单示例，展示如何获取当前时间并格式化输出。

## 实例

#include <iostream>  
#include <ctime>  
#include <iomanip>  
#include <sstream>

int main() {  
    // 获取当前时间  
    time_t now = time(NULL);

    // 将当前时间转换为本地时间  
    struct tm *local\_tm = localtime(&now);

    // 使用 strftime 格式化时间  
    char buffer[80];  
    strftime(buffer, 80, "%Y-%m-%d %H:%M:%S", local\_tm);

    // 输出当前时间  
    std::cout << "Current local time: " << buffer << std::endl;

    // 将当前时间转换为UTC时间  
    struct tm *utc\_tm = gmtime(&now);

    // 格式化UTC时间  
    strftime(buffer, 80, "%Y-%m-%d %H:%M:%S", utc\_tm);

    // 输出UTC时间  
    std::cout << "Current UTC time: " << buffer << std::endl;

    return 0;  
}

运行上述程序，你将看到类似以下的输出（具体时间取决于你运行程序的时间）：

```
Current local time: 2023-04-01 12:34:56
Current UTC time: 2023-04-01 12:34:56
```

请注意，由于时区差异，本地时间和UTC时间可能相同，也可能不同。

​`<ctime>`​ 库是 C++ 中处理时间和日期的重要工具。通过上述示例，我们可以看到如何使用 `<ctime>`​ 库来获取和格式化当前时间。这在开发需要时间信息的应用程序时非常有用，例如日志记录、定时任务等。希望这篇文章能帮助初学者更好地理解和使用 `<ctime>`​ 库。
