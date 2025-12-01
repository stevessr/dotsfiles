# C++ 标准库 <regex>

C++ 标准库中的 `<regex>`​ 头文件提供了正则表达式的功能，允许开发者使用一种非常灵活的方式来搜索、替换或分割字符串。正则表达式是一种强大的文本处理工具，广泛应用于数据验证、文本分析和模式匹配等领域。

正则表达式是一种使用单个字符串来描述、匹配一系列符合某个句法规则的字符串的模式。在 C++ 中，正则表达式通过 `<regex>`​ 库实现。

## 基本语法

### 正则表达式的基本组成

* **字符类**：如 `[abc]`​ 表示匹配 a、b 或 c 中的任意一个字符。
* **量词**：如 `*`​（零次或多次）、`+`​（一次或多次）、`?`​（零次或一次）。
* **边界匹配**：如 `^`​（行的开始）、`$`​（行的结束）。
* **分组**：使用圆括号 `()`​ 来创建一个分组。

### C++ `<regex>`​ 库的主要类和函数

* ​`std::regex`​：表示一个正则表达式对象。
* ​`std::regex_match`​：检查整个字符串是否与正则表达式匹配。
* ​`std::regex_search`​：在字符串中搜索与正则表达式匹配的部分。
* ​`std::regex_replace`​：替换字符串中与正则表达式匹配的部分。
* ​`std::sregex_iterator`​：迭代器，用于遍历所有匹配项。

## 实例

### 1. 检查字符串是否匹配正则表达式

## 实例

#include <iostream>  
#include <string>  
#include <regex>

int main() {  
    std::string text = "Hello, World!";  
    std::regex pattern("^[a-zA-Z]+, [a-zA-Z]+!$");

    if (std::regex_match(text, pattern)) {  
        std::cout << "The string matches the pattern." << std::endl;  
    } else {  
        std::cout << "The string does not match the pattern." << std::endl;  
    }

    return 0;  
}

输出结果：

```

The string matches the pattern.
```

### 2. 在字符串中搜索匹配项

## 实例

#include <iostream>  
#include <string>  
#include <regex>

int main() {  
    std::string text = "123-456-7890 and 987-654-3210";  
    std::regex pattern("\d{3}-\d{3}-\d{4}");

    std::smatch matches;  
    while (std::regex_search(text, matches, pattern)) {  
        std::cout << "Found: " << matches[0] << std::endl;  
        text = matches.suffix().str();  
    }

    return 0;  
}

输出结果：

```
Found: 123-456-7890
Found: 987-654-3210
```

### 3. 替换字符串中的匹配项

## 实例

#include <iostream>  
#include <string>  
#include <regex>

int main() {  
    std::string text = "Hello, World!";  
    std::regex pattern("World");  
    std::string replacement = "Universe";

    std::string result = std::regex_replace(text, pattern, replacement);

    std::cout << "Original: " << text << std::endl;  
    std::cout << "Modified: " << result << std::endl;

    return 0;  
}

输出结果：

```
Original: Hello, World!
Modified: Hello, Universe!
```

C++ 的 `<regex>`​ 库为处理字符串提供了强大的工具。通过上述实例，我们可以看到如何使用正则表达式来匹配、搜索和替换字符串。掌握这些基本操作后，你可以更深入地探索正则表达式的高级用法，以解决更复杂的文本处理问题。
