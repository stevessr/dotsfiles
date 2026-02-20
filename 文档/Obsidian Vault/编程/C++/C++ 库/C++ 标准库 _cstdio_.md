# C++ 标准库 <cstdio>

​`<cstdio>`​ 是 C++ 标准库中的一个头文件，它包含了 C 语言标准 I/O 库的 C++ 封装，主要用于文件的输入和输出操作。

​`<cstdio>`​ 库定义了一组用于执行输入和输出操作的函数，这些函数可以用于读写文件和控制台。

### 语法

在使用 `<cstdio>`​ 库中的函数之前，需要在 C++ 程序的顶部包含这个头文件：

```
#include <cstdio>
```

### 常用函数

​`<cstdio>`​ 库中包含了许多用于文件 I/O 的函数，以下是一些常用的函数：

* ​`fopen`​：打开文件。
* ​`fclose`​：关闭文件。
* ​`fread`​：从文件中读取数据。
* ​`fwrite`​：向文件中写入数据。
* ​`fprintf`​：向文件写入格式化输出。
* ​`fscanf`​：从文件中读取格式化输入。
* ​`fgetc`​：从文件中读取一个字符。
* ​`fputc`​：向文件写入一个字符。
* ​`fgets`​：从文件中读取一行。
* ​`fputs`​：向文件写入一行。

## 实例

1. 打开和关闭文件:

## 实例

#include <cstdio>

int main() {  
    FILE *file = fopen("example.txt", "w"); // 打开文件用于写入  
    if (file == NULL) {  
        perror("Error opening file");  
        return 1;  
    }  
    fclose(file); // 关闭文件  
    return 0;  
}

2. 向文件写入数据:

## 实例

#include <cstdio>

int main() {  
    FILE *file = fopen("example.txt", "w");  
    if (file == NULL) {  
        perror("Error opening file");  
        return 1;  
    }  
    fprintf(file, "Hello, World!n");  
    fclose(file);  
    return 0;  
}

在 "example.txt" 文件中写入了 "Hello, World!"。

3. 从文件读取数据:

## 实例

#include <cstdio>  
#include <iostream>

int main() {  
    FILE *file = fopen("example.txt", "r");  
    if (file == NULL) {  
        perror("Error opening file");  
        return 1;  
    }  
    char buffer[100];  
    while (fgets(buffer, 100, file) != NULL) {  
        std::cout << buffer;  
    }  
    fclose(file);  
    return 0;  
}

从 "example.txt" 文件中读取并输出 "Hello, World!"。

4. 使用 fscanf 和 fprintf 进行格式化输入输出:

## 实例

#include <cstdio>

int main() {  
    FILE *file = fopen("data.txt", "w");  
    if (file == NULL) {  
        perror("Error opening file");  
        return 1;  
    }  
    fprintf(file, "%d %fn", 42, 3.14159);  
    fclose(file);

    file = fopen("data.txt", "r");  
    if (file == NULL) {  
        perror("Error opening file");  
        return 1;  
    }  
    int number;  
    float pi;  
    fscanf(file, "%d %f", &number, &pi);  
    fclose(file);

    std::printf("Number: %d, Pi: %fn", number, pi);  
    return 0;  
}

输出结果：`Number: 42, Pi: 3.141590`​

### fopen 和 fclose

用于打开和关闭文件。

## 实例

#include <cstdio>

int main() {  
    FILE* file = fopen("example.txt", "r");  
    if (file) {  
        // 文件操作  
        fclose(file);  
    } else {  
        // 处理错误  
    }  
    return 0;  
}

### fread 和 fwrite

用于从文件中读取和写入数据。

## 实例

#include <cstdio>

int main() {  
    FILE* file = fopen("example.bin", "wb");  
    if (file) {  
        int data = 12345;  
        fwrite(&data, sizeof(data), 1, file);  
        fclose(file);  
    }  
    return 0;  
}

### fseek 和 ftell

用于在文件中移动文件指针和获取文件指针的位置。

## 实例

#include <cstdio>

int main() {  
    FILE* file = fopen("example.txt", "r");  
    if (file) {  
        fseek(file, 0, SEEK_END); // 移动到文件末尾  
        long size = ftell(file);  // 获取文件大小  
        fclose(file);  
    }  
    return 0;  
}

### fflush

刷新文件流，将缓冲区中的数据写入文件。

## 实例

#include <cstdio>

int main() {  
    FILE* file = fopen("example.txt", "w");  
    if (file) {  
        fputs("Hello, World!", file);  
        fflush(file); // 确保数据立即写入文件  
        fclose(file);  
    }  
    return 0;  
}

### printf 和 fprintf

用于格式化输出到标准输出或文件。

## 实例

#include <cstdio>

int main() {  
    int value = 42;  
    printf("Value: %d\n", value); // 输出到标准输出

    FILE* file = fopen("output.txt", "w");  
    if (file) {  
        fprintf(file, "Value: %d\n", value); // 输出到文件  
        fclose(file);  
    }  
    return 0;  
}

### scanf 和 fscanf

用于格式化输入从标准输入或文件。

## 实例

#include <cstdio>

int main() {  
    int value;  
    scanf("%d", &value); // 从标准输入读取

    FILE* file = fopen("input.txt", "r");  
    if (file) {  
        fscanf(file, "%d", &value); // 从文件读取  
        fclose(file);  
    }  
    return 0;  
}

### sprintf 和 sscanf

用于格式化输出到字符串和从字符串中读取。

## 实例

#include <cstdio>

int main() {  
    char buffer[50];  
    int value = 42;  
    sprintf(buffer, "Value: %d", value); // 输出到字符串

    int readValue;  
    sscanf(buffer, "Value: %d", &readValue); // 从字符串读取

    return 0;  
}

### fgets 和 fputs

用于从文件中读取字符串和写入字符串到文件。

## 实例

#include <cstdio>

int main() {  
    FILE* file = fopen("example.txt", "r");  
    if (file) {  
        char buffer[100];  
        if (fgets(buffer, sizeof(buffer), file)) {  
            // 读取成功  
        }  
        fclose(file);  
    }

    file = fopen("example.txt", "w");  
    if (file) {  
        fputs("Hello, World!\n", file); // 写入字符串到文件  
        fclose(file);  
    }  
    return 0;  
}

### getc 和 putc

用于从文件中读取字符和写入字符到文件。

## 实例

#include <cstdio>

int main() {  
    FILE* file = fopen("example.txt", "r");  
    if (file) {  
        int c = getc(file); // 读取一个字符  
        fclose(file);  
    }

    file = fopen("example.txt", "w");  
    if (file) {  
        putc('A', file); // 写入一个字符  
        fclose(file);  
    }  
    return 0;  
}

### feof 和 ferror

用于检测文件结束和文件错误。

## 实例

#include <cstdio>

int main() {  
    FILE* file = fopen("example.txt", "r");  
    if (file) {  
        while (!feof(file)) {  
            int c = getc(file);  
            if (ferror(file)) {  
                // 处理错误  
                break;  
            }  
            // 处理字符  
        }  
        fclose(file);  
    }  
    return 0;  
}

### EOF

表示文件结束标志。

## 实例

#include <cstdio>

int main() {  
    FILE* file = fopen("example.txt", "r");  
    if (file) {  
        int c;  
        while ((c = getc(file)) != EOF) {  
            // 处理字符  
        }  
        fclose(file);  
    }  
    return 0;  
}

### NULL

表示空指针。

## 实例

#include <cstdio>

int main() {  
    FILE* file = fopen("example.txt", "r");  
    if (file == NULL) {  
        // 处理文件打开错误  
    } else {  
        fclose(file);  
    }  
    return 0;  
}

## 注意事项

* 使用 `<cstdio>`​ 库时，需要确保正确处理文件打开和关闭，避免资源泄露。
* 在使用 `fopen`​ 时，需要提供正确的文件路径和模式。
* 使用 `fprintf`​ 和 `fscanf`​ 时，格式化字符串应该与变量类型匹配。
