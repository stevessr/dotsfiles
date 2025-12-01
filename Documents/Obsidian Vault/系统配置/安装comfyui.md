[toc]

## 先安装python

```cmd
winget install Python.Python.3.11
```
或者去[Python Release Python 3.11.9 | Python.org](https://www.python.org/downloads/release/python-3119/)下载python3.11并安装
## 如果没有git就需要安装git
```cmd
winget install git.git
```
或者去[Git - Downloads (git-scm.com)](https://git-scm.com/downloads)下载并安装
## 如果没有visual studio 的build tools也要安装visual studio的build tools
```cmd
winget install Microsoft.VisualStudio.2022.BuildTools
```
或者在[下载 Visual Studio Tools - 免费安装 Windows、Mac、Linux (microsoft.com)](https://visualstudio.microsoft.com/zh-hans/downloads/)下载

## 如果没有rust也要安装rust
```cmd
winget install Rustlang.Rust.MSVC   
#if you like GNU
#try this
#winget install Rustlang.Rust.GNU
```
或者在[Install Rust - Rust Programming Language (rust-lang.org)](https://www.rust-lang.org/tools/install)下载并安装
## 再克隆comfyui的仓库

```cmd
#找一个足够大的地方打开cmd
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
main.py
#然后会自动配好
```
在这里解压huishi.zip
打开A启动器
在绘世里设置->一般配置->配置模式 改成专家
在高级选项->环境维护->安装Pytorch 中 安装
Torch 2 ． 4 ． 1 (CUDA 12 ． 4 ）+ xFormers 0 ， 0 ． 28 ． post1
在高级选项->环境维护->原生组件管理 中 安装/同步
```raw
FFmpeg 
Microsoft Visual C++ Redistributable 
Visual Studio Build Tools 
CUDA Toolkit 
CMake 
Ninja 
```
打开ComfyUI\.msvc\VC\Tools\MSVC\14.41.34120\bin\Hostx64\x64 
把这个位置加入系统path

在版本管理中 安装所需插件
在模型管理中 安装想要的模型
flux.1: [Flux Examples | ComfyUI_examples (comfyanonymous.github.io)](https://comfyanonymous.github.io/ComfyUI_examples/flux/)
SD: [stabilityai (Stability AI) (huggingface.co)](https://huggingface.co/stabilityai)
可以使用tensorRT加速生成（推理）
### 安装tensorRT
使用main.py启动
在节点管理器中选择tensorRT并安装

#### tips
Watt Toolkit 实验性支持Huggface 归类在Github下