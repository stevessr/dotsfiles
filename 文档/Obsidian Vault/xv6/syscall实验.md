# 关于gdb调试
先打开gdb
`gdb [pdb library]`
## 连接远程
`target remote [ip]:port`
## 添加断点
`b [*visual memory offset|function name]`
## 控制流
### 下一句
`n`
### 继续到下一次断点
`c`
### 继续到下一个调用函数
`s`
## terminal UI 修改
`layout [UI target]`
### 源码视图
`layout src`
### 汇编视图
`layout asm`
### 视图切割
`layout split`
### 切换视图
`layout next/prev`
### 寄存器视图
`layout regs`