---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/PHP principle/OPcache/","title":"OPcache","tags":["flashcards"],"noteIcon":"","created":"2026-05-21T11:11:59.000+08:00","updated":"2026-05-21T11:11:59.000+08:00","dg-note-properties":{"title":"OPcache","tags":["flashcards"],"reference linking":null}}
---

# 一、引言：重要概念
## OPcache（Optimized PHP Code Accelerator） 
是PHP内置的字节码缓存扩展，通过缓存PHP脚本的编译结果（字节码），显著减少重复解析和编译的开销，从而提升PHP应用的执行效率。它是现代PHP性能优化的核心组件之一。
## 解释型、编译型语言
### 解释型语言（Interpreted Language）
#### 定义  
解释型语言的程序在运行时由 **解释器（Interpreter）** 逐行读取、解析并执行，**不生成独立的机器码文件**，而是直接翻译并执行源代码。  
例如：Python、JavaScript、Ruby、PHP。
#### 核心特点  
- **逐行执行**：每执行一行代码，解释器实时翻译并运行，无需预先编译。  
- **跨平台性**：只要目标平台安装了对应的解释器，同一份源代码可在不同操作系统上运行。  
- **灵活性高**：代码修改后可立即运行，无需重新编译，适合快速开发和调试。  
- **性能较低**：每次运行都需要解释器参与，效率通常低于编译型语言。  
- **内存管理**：多数由解释器自动处理（如垃圾回收机制），开发者无需手动管理内存。
#### 典型例子  
- **Python**：通过 `python` 命令直接运行 `.py` 文件。  
- **JavaScript**：在浏览器中通过 V8 引擎（如 Chrome）或 Node.js 解释执行。  
- **PHP**：通过 Zend 引擎解释执行脚本。
### 编译型语言（Compiled Language）
#### 定义  
编译型语言的程序在运行前必须通过 **编译器（Compiler）** 将源代码转换为 **机器码（二进制文件）**，生成独立的可执行文件（如 `.exe`、`.out`）。  
例如：C、C++、Go、Rust。
#### 核心特点  
- **一次性编译**：编译后生成可直接运行的机器码，无需依赖源代码或编译器。  
- **执行速度快**：机器码直接由 CPU 执行，无需翻译过程，性能更高。  
- **跨平台性差**：编译后的可执行文件依赖特定操作系统和硬件架构（如 x86、ARM）。  
- **内存管理**：通常需要开发者手动管理内存（如 C/C++ 的 `malloc/free`）。  
- **安全性高**：代码以二进制形式分发，反编译难度较大。
#### 典型例子  
- **C/C++**：通过 `gcc` 或 `clang` 编译生成可执行文件。  
- **Go**：编译为独立的二进制文件，支持跨平台编译（如 `GOOS=linux GOARCH=amd64 go build`）。  
- **Rust**：编译为高性能的机器码，适用于系统级开发。
### 总结
| **特性**    | **解释型语言**             | **编译型语言**          |
| --------- | --------------------- | ------------------ |
| **执行方式**  | 运行时逐行解释执行             | 先编译为机器码，后直接运行      |
| **可执行文件** | 无（依赖解释器）              | 有（独立二进制文件）         |
| **跨平台性**  | 高（依赖解释器）              | 低（依赖操作系统/架构）       |
| **执行速度**  | 较慢（需逐行翻译）             | 快（直接机器码执行）         |
| **内存管理**  | 自动（垃圾回收）              | 手动或自动（如 Go 的垃圾回收）  |
| **开发效率**  | 高（无需编译，修改即运行）         | 低（需频繁编译）           |
| **典型语言**  | Python、JavaScript、PHP | C、C++、Go、Rust、JAVA |
## 解释器、编译器、混合模式
### 解释器（Interpreter）
#### 定义：
逐条将源代码翻译成机器指令并立即执行，**不生成中间或最终的机器码文件**。
#### 流程：
1. **解释阶段**：逐行读取源代码，实时翻译成可执行指令。
2. **执行阶段**：翻译后的指令直接由计算机执行。
#### 特点：
- **执行速度慢**：每条语句需重复翻译，效率低于编译型程序。
- **依赖解释器**：程序运行时必须有解释器存在。
#### 典型解释器  
- **Python解释器**：通过 `python` 命令直接运行 `.py` 文件。  
- **JavaScript引擎（V8）**：Chrome 浏览器中将 JavaScript 转换为机器码。  
- **PHP解释器（Zend Engine）**：处理 PHP 脚本并生成 Opcode。
### 编译器（Compiler）
####  定义
将源程序的**每一条语句**编译成**机器语言代码**，并保存为**二进制文件**（可执行文件或目标代码）。
####  流程
1. **编译阶段**：将源代码转换为与硬件直接兼容的机器码。
2. **执行阶段**：运行时计算机直接加载二进制文件，无需再次翻译。
####  特点
- **执行速度快**：机器码可被硬件直接执行，无需额外翻译。
- **独立性**：生成的二进制文件可脱离编译环境运行。
#### 典型编译器  
- **GCC**：支持 C、C++、Go 等语言的编译。  
- **Clang**：基于 LLVM 的高效编译器。  
- **javac**：Java 编译器，生成字节码 `.class` 文件。
### 混合模式：现代语言的突破
随着技术发展，许多语言结合了编译和解释的特性：  
1. **JIT（即时编译）**：  
   - 如 Java 的 JVM、JavaScript 的 V8 引擎，先将代码编译为中间字节码，再在运行时动态编译为机器码（如热点代码优化）。  
2. **预编译中间代码**：  
   - 如 C# 的 CLR、Java 的 JVM，将代码编译为平台无关的字节码，再由虚拟机解释或 JIT 编译执行。
### 总结
| 特性      | 编译器                          | 解释器                                      | 混合模式（如 JIT）                          |
|---------|------------------------------|------------------------------------------|--------------------------------------|
| 定义      | 将源代码转换为机器码，生成可执行文件           | 逐行翻译并执行，不生成机器码文件                         | 先编译为中间代码，运行时动态编译优化代码                 |
| 执行方式    | 一次性编译，之后直接执行机器码              | 逐行解释执行，无中间文件生成                           | 预编译为中间代码（如字节码），运行时优化执行               |
| 执行速度    | 快，机器码直接执行                    | 慢，逐行翻译开销大                                | 初始慢，但通过JIT编译高频代码后接近编译速度              |
| 依赖环境    | 可执行文件独立于编译器运行                | 需要解释器存在才能运行                              | 需要虚拟机（如JVM、CLR）或运行时环境                |
| 错误调试    | 编译期报错，可能难以定位复杂错误             | 运行时逐行报错，调试更直观                            | 运行时错误调试，结合编译期和运行期反馈                  |
| 跨平台性    | 生成特定平台代码（需重新编译）              | 通过解释器实现跨平台（需对应解释器）                       | 中间代码（如字节码）跨平台，依赖虚拟机                  |
| 生成的文件类型 | 可执行文件（如 .exe, .out）          | 无独立可执行文件                                 | 中间代码文件（如 Java 的 .class）              |
| 典型例子    | GCC（C/C++）、Clang、javac（Java） | Python解释器、JavaScript V8（解释模式）、PHP Zend引擎 | Java JVM、C# CLR、JavaScript V8（JIT模式） |
# 二、基本原理

## 1. PHP脚本的执行流程  
- **CLI模式：**  
	每次执行脚本均需完整流程：  
	`初始化Zend引擎 → 加载扩展 → 读取脚本 → 词法/语法分析 → 生成AST → 编译为Opcode → 执行Opcode`
- **php-fpm模式：**  
	初始化（Zend引擎加载扩展）仅在服务启动时执行，后续请求仅需：  
	`读取脚本 → 词法/语法分析/生成语法生成树(AST) → 编译Opcode → 执行`
## 2. OPcache的优化  
- **缓存字节码**：首次执行后，OPcache将编译后的字节码存储在共享内存中。  
- **后续请求**：直接使用缓存的字节码，跳过解析和编译步骤，显著降低CPU和内存消耗。  
**示例**：  
假设一个PHP文件`index.php`包含以下代码：  
```php
echo "Hello World!";
```  
- **首次请求**：PHP解释器解析并编译为Opcode，缓存到OPcache。  
- **后续请求**：直接执行缓存的Opcode，无需重新解析。
## 3. OPcache工作流程示意图
<div class="excalidraw-svg"><svg version="1.1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 820 3780" width="820" height="3780" class="excalidraw-svg" style="max-width: 100%; height: auto;"><!-- svg-source:excalidraw --><metadata/><defs><style class="style-fonts">      @font-face { font-family: Comic Shanns; src: url(data:font/woff2;base64,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); }</style></defs><rect x="0" y="0" width="820" height="3780" fill="#ffffff"/><g stroke-linecap="round" transform="translate(10 250) rotate(0 80 80)"><path d="M32 0 C69.9 0, 107.8 0, 128 0 M32 0 C56.46 0, 80.92 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 64.92, 160 97.84, 160 128 M160 32 C160 55.92, 160 79.84, 160 128 M160 128 C160 149.33, 149.33 160, 128 160 M160 128 C160 149.33, 149.33 160, 128 160 M128 160 C107.26 160, 86.52 160, 32 160 M128 160 C91.55 160, 55.1 160, 32 160 M32 160 C10.67 160, 0 149.33, 0 128 M32 160 C10.67 160, 0 149.33, 0 128 M0 128 C0 94.34, 0 60.69, 0 32 M0 128 C0 96.94, 0 65.88, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(44.800018310546875 320) rotate(0 45.199981689453125 10)"><text x="45.199981689453125" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">请求PHP脚本</text></g><g stroke-linecap="round" transform="translate(10 570) rotate(0 80 80)"><path d="M32 0 C66.2 0, 100.4 0, 128 0 M32 0 C57.51 0, 83.01 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 62.31, 160 92.62, 160 128 M160 32 C160 69.6, 160 107.2, 160 128 M160 128 C160 149.33, 149.33 160, 128 160 M160 128 C160 149.33, 149.33 160, 128 160 M128 160 C97.88 160, 67.77 160, 32 160 M128 160 C107.61 160, 87.22 160, 32 160 M32 160 C10.67 160, 0 149.33, 0 128 M32 160 C10.67 160, 0 149.33, 0 128 M0 128 C0 104.92, 0 81.83, 0 32 M0 128 C0 97.73, 0 67.46, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(19.200042724609375 630) rotate(0 70.79995727539062 20)"><text x="70.79995727539062" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">OPcache检查共享内</text><text x="70.79995727539062" y="34" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">存</text></g><g stroke-linecap="round" transform="translate(10 1850) rotate(0 80 83.75)"><path d="M32 0 C69.73 0, 107.46 0, 128 0 M32 0 C63.96 0, 95.91 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 57.33, 160 82.66, 160 135.5 M160 32 C160 69.2, 160 106.39, 160 135.5 M160 135.5 C160 156.83, 149.33 167.5, 128 167.5 M160 135.5 C160 156.83, 149.33 167.5, 128 167.5 M128 167.5 C92.66 167.5, 57.32 167.5, 32 167.5 M128 167.5 C107.05 167.5, 86.09 167.5, 32 167.5 M32 167.5 C10.67 167.5, 0 156.83, 0 135.5 M32 167.5 C10.67 167.5, 0 156.83, 0 135.5 M0 135.5 C0 104.92, 0 74.34, 0 32 M0 135.5 C0 111.37, 0 87.25, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(34 1923.75) rotate(0 56 10)"><text x="56" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">直接执行字节码</text></g><g stroke-linecap="round" transform="translate(10 2170) rotate(0 80 80)"><path d="M32 0 C51.83 0, 71.66 0, 128 0 M32 0 C51.56 0, 71.12 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 66.64, 160 101.28, 160 128 M160 32 C160 58.94, 160 85.89, 160 128 M160 128 C160 149.33, 149.33 160, 128 160 M160 128 C160 149.33, 149.33 160, 128 160 M128 160 C101.58 160, 75.16 160, 32 160 M128 160 C99.18 160, 70.36 160, 32 160 M32 160 C10.67 160, 0 149.33, 0 128 M32 160 C10.67 160, 0 149.33, 0 128 M0 128 C0 92.47, 0 56.94, 0 32 M0 128 C0 107.66, 0 87.33, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(58 2240) rotate(0 32 10)"><text x="32" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">返回结果</text></g><g stroke-linecap="round" transform="translate(330 1050) rotate(0 80 80)"><path d="M101.25 20.25 C113.88 33.54, 126.51 46.83, 139.75 60.75 M101.25 20.25 C115.52 35.26, 129.79 50.27, 139.75 60.75 M139.75 60.75 C160 81, 160 81, 139.75 101.25 M139.75 60.75 C160 81, 160 81, 139.75 101.25 M139.75 101.25 C130.97 110.03, 122.19 118.81, 101.25 139.75 M139.75 101.25 C128.06 112.94, 116.36 124.64, 101.25 139.75 M101.25 139.75 C81 160, 81 160, 60.75 139.75 M101.25 139.75 C81 160, 81 160, 60.75 139.75 M60.75 139.75 C45.34 125.1, 29.93 110.45, 20.25 101.25 M60.75 139.75 C46.13 125.85, 31.51 111.95, 20.25 101.25 M20.25 101.25 C0 81, 0 81, 20.25 60.75 M20.25 101.25 C0 81, 0 81, 20.25 60.75 M20.25 60.75 C29.02 51.98, 37.79 43.21, 60.75 20.25 M20.25 60.75 C34.81 46.19, 49.36 31.64, 60.75 20.25 M60.75 20.25 C81 0, 81 0, 101.25 20.25 M60.75 20.25 C81 0, 81 0, 101.25 20.25" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(389.6000061035156 1110) rotate(0 20.399993896484375 20)"><text x="20.399993896484375" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">文件</text><text x="20.399993896484375" y="34" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">修改?</text></g><g stroke-linecap="round" transform="translate(330 1370) rotate(0 80 83.75)"><path d="M32 0 C60.87 0, 89.73 0, 128 0 M32 0 C68.77 0, 105.55 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 71.76, 160 111.53, 160 135.5 M160 32 C160 71.81, 160 111.62, 160 135.5 M160 135.5 C160 156.83, 149.33 167.5, 128 167.5 M160 135.5 C160 156.83, 149.33 167.5, 128 167.5 M128 167.5 C99.89 167.5, 71.77 167.5, 32 167.5 M128 167.5 C97.95 167.5, 67.91 167.5, 32 167.5 M32 167.5 C10.67 167.5, 0 156.83, 0 135.5 M32 167.5 C10.67 167.5, 0 156.83, 0 135.5 M0 135.5 C0 96.14, 0 56.77, 0 32 M0 135.5 C0 112.06, 0 88.63, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(378 1443.75) rotate(0 32 10)"><text x="32" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">重新编译</text></g><g stroke-linecap="round" transform="translate(330 1690) rotate(0 80 80)"><path d="M32 0 C55.78 0, 79.56 0, 128 0 M32 0 C52.96 0, 73.92 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 62.79, 160 93.57, 160 128 M160 32 C160 60.99, 160 89.97, 160 128 M160 128 C160 149.33, 149.33 160, 128 160 M160 128 C160 149.33, 149.33 160, 128 160 M128 160 C106.72 160, 85.43 160, 32 160 M128 160 C91.82 160, 55.64 160, 32 160 M32 160 C10.67 160, 0 149.33, 0 128 M32 160 C10.67 160, 0 149.33, 0 128 M0 128 C0 98.71, 0 69.42, 0 32 M0 128 C0 107.38, 0 86.75, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(378 1760) rotate(0 32 10)"><text x="32" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">更新缓存</text></g><g stroke-linecap="round" transform="translate(330 2170) rotate(0 80 80)"><path d="M32 0 C51.26 0, 70.53 0, 128 0 M32 0 C68.93 0, 105.86 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 65.32, 160 98.65, 160 128 M160 32 C160 66.79, 160 101.57, 160 128 M160 128 C160 149.33, 149.33 160, 128 160 M160 128 C160 149.33, 149.33 160, 128 160 M128 160 C90.65 160, 53.31 160, 32 160 M128 160 C101.74 160, 75.48 160, 32 160 M32 160 C10.67 160, 0 149.33, 0 128 M32 160 C10.67 160, 0 149.33, 0 128 M0 128 C0 103.08, 0 78.17, 0 32 M0 128 C0 91.98, 0 55.95, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(370 2240) rotate(0 40 10)"><text x="40" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">执行字节码</text></g><g stroke-linecap="round" transform="translate(650 1370) rotate(0 80 80)"><path d="M32 0 C69.22 0, 106.44 0, 128 0 M32 0 C53.44 0, 74.87 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 58.19, 160 84.38, 160 128 M160 32 C160 65.84, 160 99.68, 160 128 M160 128 C160 149.33, 149.33 160, 128 160 M160 128 C160 149.33, 149.33 160, 128 160 M128 160 C97.31 160, 66.63 160, 32 160 M128 160 C94.33 160, 60.67 160, 32 160 M32 160 C10.67 160, 0 149.33, 0 128 M32 160 C10.67 160, 0 149.33, 0 128 M0 128 C0 97.43, 0 66.86, 0 32 M0 128 C0 105.73, 0 83.45, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(690 1440) rotate(0 40 10)"><text x="40" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">解析并编译</text></g><g stroke-linecap="round" transform="translate(650 1690) rotate(0 80 80)"><path d="M32 0 C64.92 0, 97.84 0, 128 0 M32 0 C55.92 0, 79.84 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 52.74, 160 73.48, 160 128 M160 32 C160 68.45, 160 104.9, 160 128 M160 128 C160 149.33, 149.33 160, 128 160 M160 128 C160 149.33, 149.33 160, 128 160 M128 160 C94.34 160, 60.69 160, 32 160 M128 160 C96.94 160, 65.88 160, 32 160 M32 160 C10.67 160, 0 149.33, 0 128 M32 160 C10.67 160, 0 149.33, 0 128 M0 128 C0 92.31, 0 56.63, 0 32 M0 128 C0 106.51, 0 85.02, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(690 1760) rotate(0 40 10)"><text x="40" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">缓存字节码</text></g><g stroke-linecap="round"><g transform="translate(89.90000000000009 94.99996885812061) rotate(0 0 72.5000155709397)"><path d="M0 0 C0 45.78, 0 91.56, 0 145 M0 0 C0 56.79, 0 113.59, 0 145" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.90000000000009 94.99996885812061) rotate(0 0 72.5000155709397)"><path d="M0 145 L-6.34 131.41 L6.34 131.41 L0 145" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 145 C-2 140.71, -4 136.42, -6.34 131.41 M0 145 C-2.48 139.68, -4.97 134.35, -6.34 131.41 M-6.34 131.41 C-2.27 131.41, 1.79 131.41, 6.34 131.41 M-6.34 131.41 C-2.51 131.41, 1.32 131.41, 6.34 131.41 M6.34 131.41 C4.59 135.15, 2.85 138.89, 0 145 M6.34 131.41 C4.44 135.49, 2.53 139.57, 0 145 M0 145 C0 145, 0 145, 0 145 M0 145 C0 145, 0 145, 0 145" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(89.90000000000009 420) rotate(0 0 68.33333333333337)"><path d="M0 0 C0 51.97, 0 103.94, 0 136.67 M0 0 C0 45.8, 0 91.6, 0 136.67" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.90000000000009 420) rotate(0 0 68.33333333333337)"><path d="M0 136.67 L-6.34 123.07 L6.34 123.07 L0 136.67" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 136.67 C-2.41 131.5, -4.82 126.33, -6.34 123.07 M0 136.67 C-2.12 132.11, -4.25 127.56, -6.34 123.07 M-6.34 123.07 C-1.94 123.07, 2.46 123.07, 6.34 123.07 M-6.34 123.07 C-1.75 123.07, 2.85 123.07, 6.34 123.07 M6.34 123.07 C4.08 127.91, 1.82 132.75, 0 136.67 M6.34 123.07 C3.89 128.33, 1.44 133.58, 0 136.67 M0 136.67 C0 136.67, 0 136.67, 0 136.67 M0 136.67 C0 136.67, 0 136.67, 0 136.67" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(89.90000000000032 2022.5) rotate(0 -1.1368683772161603e-13 71.24999999999999)"><path d="M0 0 C0 47.46, 0 94.91, 0 142.5 M0 0 C0 41.61, 0 83.21, 0 142.5" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.90000000000032 2022.5) rotate(0 -1.1368683772161603e-13 71.24999999999999)"><path d="M0 142.5 L-6.34 128.91 L6.34 128.91 L0 142.5" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 142.5 C-2.11 137.97, -4.22 133.45, -6.34 128.91 M0 142.5 C-1.85 138.53, -3.7 134.56, -6.34 128.91 M-6.34 128.91 C-2.36 128.91, 1.62 128.91, 6.34 128.91 M-6.34 128.91 C-3.65 128.91, -0.95 128.91, 6.34 128.91 M6.34 128.91 C4.93 131.92, 3.53 134.93, 0 142.5 M6.34 128.91 C3.95 134.03, 1.56 139.15, 0 142.5 M0 142.5 C0 142.5, 0 142.5, 0 142.5 M0 142.5 C0 142.5, 0 142.5, 0 142.5" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(89.90000000000009 3618) rotate(0 0 33.50001557093958)"><path d="M0 0 C0 15.28, 0 30.56, 0 67 M0 0 C0 20.35, 0 40.7, 0 67" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.90000000000009 3618) rotate(0 0 33.50001557093958)"><path d="M0 67 L-6.34 53.41 L6.34 53.41 L0 67" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 67 C-1.45 63.9, -2.89 60.8, -6.34 53.41 M0 67 C-1.93 62.87, -3.85 58.74, -6.34 53.41 M-6.34 53.41 C-2.15 53.41, 2.04 53.41, 6.34 53.41 M-6.34 53.41 C-2.27 53.41, 1.8 53.41, 6.34 53.41 M6.34 53.41 C3.87 58.69, 1.41 63.98, 0 67 M6.34 53.41 C4.61 57.12, 2.87 60.84, 0 67 M0 67 C0 67, 0 67, 0 67 M0 67 C0 67, 0 67, 0 67" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g mask="url(#mask-MU3V08kTsvu9bTFv-L7QM)" stroke-linecap="round"><g transform="translate(170 970) rotate(0 119.95000443577283 37.8026638694318)"><path d="M0 0 C86.02 0, 172.04 0, 223.9 0 M0 0 C86.12 0, 172.24 0, 223.9 0 M223.9 0 C234.57 0, 239.9 5.33, 239.9 16 M223.9 0 C234.57 0, 239.9 5.33, 239.9 16 M239.9 16 C239.9 33.46, 239.9 50.91, 239.9 75.61 M239.9 16 C239.9 34.66, 239.9 53.31, 239.9 75.61" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(170 970) rotate(0 119.95000443577283 37.8026638694318)"><path d="M239.9 75.61 L233.56 62.01 L246.24 62.01 L239.9 75.61" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M239.9 75.61 C237.46 70.38, 235.03 65.16, 233.56 62.01 M239.9 75.61 C237.46 70.38, 235.02 65.15, 233.56 62.01 M233.56 62.01 C238.23 62.01, 242.9 62.01, 246.24 62.01 M233.56 62.01 C236.33 62.01, 239.1 62.01, 246.24 62.01 M246.24 62.01 C244.21 66.36, 242.19 70.7, 239.9 75.61 M246.24 62.01 C244.02 66.78, 241.79 71.55, 239.9 75.61 M239.9 75.61 C239.9 75.61, 239.9 75.61, 239.9 75.61 M239.9 75.61 C239.9 75.61, 239.9 75.61, 239.9 75.61" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask id="mask-MU3V08kTsvu9bTFv-L7QM" maskUnits="userSpaceOnUse" x="0" y="0" width="509.90000887154565" height="1145.6053277388637"><rect x="0" y="0" fill="#fff" width="509.90000887154565" height="1145.6053277388637"/><rect x="396.90000887154565" y="955" fill="#000" width="26" height="30" opacity="1"/></mask><g transform="translate(401.90000887154565 960) rotate(0 -111.95000443577283 47.8026638694318)"><text x="8" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">否</text></g><g stroke-linecap="round"><g transform="translate(409.9000000000003 1542.5) rotate(0 -1.1368683772161603e-13 71.25)"><path d="M0 0 C0 49.47, 0 98.93, 0 142.5 M0 0 C0 51.63, 0 103.27, 0 142.5" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(409.9000000000003 1542.5) rotate(0 -1.1368683772161603e-13 71.25)"><path d="M0 142.5 L-6.34 128.91 L6.34 128.91 L0 142.5" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 142.5 C-2.2 137.78, -4.4 133.06, -6.34 128.91 M0 142.5 C-2.3 137.57, -4.59 132.65, -6.34 128.91 M-6.34 128.91 C-1.82 128.91, 2.69 128.91, 6.34 128.91 M-6.34 128.91 C-1.44 128.91, 3.46 128.91, 6.34 128.91 M6.34 128.91 C4.82 132.17, 3.29 135.44, 0 142.5 M6.34 128.91 C4.34 133.19, 2.34 137.48, 0 142.5 M0 142.5 C0 142.5, 0 142.5, 0 142.5 M0 142.5 C0 142.5, 0 142.5, 0 142.5" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(409.9000000000001 1855) rotate(0 0 155)"><path d="M0 0 C0 84.56, 0 169.13, 0 310 M0 0 C0 109.28, 0 218.55, 0 310" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(409.9000000000001 1855) rotate(0 0 155)"><path d="M0 310 L-6.34 296.41 L6.34 296.41 L0 310" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 310 C-1.73 306.29, -3.46 302.58, -6.34 296.41 M0 310 C-2.23 305.21, -4.47 300.42, -6.34 296.41 M-6.34 296.41 C-1.52 296.41, 3.31 296.41, 6.34 296.41 M-6.34 296.41 C-1.76 296.41, 2.82 296.41, 6.34 296.41 M6.34 296.41 C4.57 300.2, 2.8 303.98, 0 310 M6.34 296.41 C3.82 301.82, 1.29 307.23, 0 310 M0 310 C0 310, 0 310, 0 310 M0 310 C0 310, 0 310, 0 310" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g mask="url(#mask-poq_K86hcGYH7Yfh0jiJk)" stroke-linecap="round"><g transform="translate(494.68334383851584 1129.906990655521) rotate(0 117.60832808074213 117.54650467223956)"><path d="M0 0 C47.36 0, 94.72 0, 219.22 0 M0 0 C83.23 0, 166.47 0, 219.22 0 M219.22 0 C229.88 0, 235.22 5.33, 235.22 16 M219.22 0 C229.88 0, 235.22 5.33, 235.22 16 M235.22 16 C235.22 92.81, 235.22 169.63, 235.22 235.09 M235.22 16 C235.22 86.88, 235.22 157.77, 235.22 235.09" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(494.68334383851584 1129.906990655521) rotate(0 117.60832808074213 117.54650467223956)"><path d="M235.22 235.09 L228.88 221.5 L241.56 221.5 L235.22 235.09" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M235.22 235.09 C233.85 232.16, 232.48 229.22, 228.88 221.5 M235.22 235.09 C232.81 229.93, 230.4 224.77, 228.88 221.5 M228.88 221.5 C232.59 221.5, 236.3 221.5, 241.56 221.5 M228.88 221.5 C232.85 221.5, 236.81 221.5, 241.56 221.5 M241.56 221.5 C239.68 225.51, 237.81 229.53, 235.22 235.09 M241.56 221.5 C240.08 224.67, 238.6 227.84, 235.22 235.09 M235.22 235.09 C235.22 235.09, 235.22 235.09, 235.22 235.09 M235.22 235.09 C235.22 235.09, 235.22 235.09, 235.22 235.09" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask id="mask-poq_K86hcGYH7Yfh0jiJk" maskUnits="userSpaceOnUse" x="0" y="0" width="829.9000000000001" height="1465"><rect x="0" y="0" fill="#fff" width="829.9000000000001" height="1465"/><rect x="716.9000000000003" y="1114.906990655521" fill="#000" width="26" height="30" opacity="1"/></mask><g transform="translate(721.9000000000001 1119.906990655521) rotate(0 -109.60832808074213 127.54650467223956)"><text x="8" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">否</text></g><g stroke-linecap="round"><g transform="translate(729.9000000000001 1535) rotate(0 0 75)"><path d="M0 0 C0 47.06, 0 94.11, 0 150 M0 0 C0 31.86, 0 63.72, 0 150" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(729.9000000000001 1535) rotate(0 0 75)"><path d="M0 150 L-6.34 136.41 L6.34 136.41 L0 150" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 150 C-1.99 145.74, -3.98 141.47, -6.34 136.41 M0 150 C-1.35 147.11, -2.69 144.23, -6.34 136.41 M-6.34 136.41 C-3.53 136.41, -0.72 136.41, 6.34 136.41 M-6.34 136.41 C-1.56 136.41, 3.22 136.41, 6.34 136.41 M6.34 136.41 C3.99 141.44, 1.65 146.47, 0 150 M6.34 136.41 C5 139.29, 3.65 142.17, 0 150 M0 150 C0 150, 0 150, 0 150 M0 150 C0 150, 0 150, 0 150" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(325 2249.9) rotate(0 -75 0)"><path d="M0 0 C-55.22 0, -110.44 0, -150 0 M0 0 C-32.74 0, -65.48 0, -150 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(325 2249.9) rotate(0 -75 0)"><path d="M-150 0 L-136.41 -6.34 L-136.41 6.34 L-150 0" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M-150 0 C-145 -2.33, -139.99 -4.67, -136.41 -6.34 M-150 0 C-147.03 -1.38, -144.07 -2.77, -136.41 -6.34 M-136.41 -6.34 C-136.41 -2.29, -136.41 1.77, -136.41 6.34 M-136.41 -6.34 C-136.41 -1.89, -136.41 2.55, -136.41 6.34 M-136.41 6.34 C-139.35 4.97, -142.29 3.59, -150 0 M-136.41 6.34 C-141.29 4.06, -146.18 1.78, -150 0 M-150 0 C-150 0, -150 0, -150 0 M-150 0 C-150 0, -150 0, -150 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round" transform="translate(10 2490) rotate(0 80 80)"><path d="M32 0 C63.13 0, 94.25 0, 128 0 M32 0 C56.87 0, 81.75 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 59.69, 160 87.38, 160 128 M160 32 C160 52.88, 160 73.76, 160 128 M160 128 C160 149.33, 149.33 160, 128 160 M160 128 C160 149.33, 149.33 160, 128 160 M128 160 C104.42 160, 80.84 160, 32 160 M128 160 C106.44 160, 84.88 160, 32 160 M32 160 C10.67 160, 0 149.33, 0 128 M32 160 C10.67 160, 0 149.33, 0 128 M0 128 C0 99.8, 0 71.6, 0 32 M0 128 C0 103.47, 0 78.94, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(42 2560) rotate(0 48 10)"><text x="48" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">缓存持续有效</text></g><g stroke-linecap="round" transform="translate(10 2810) rotate(0 80 80)"><path d="M101.25 20.25 C115.85 35.61, 130.44 50.96, 139.75 60.75 M101.25 20.25 C113.16 32.78, 125.08 45.32, 139.75 60.75 M139.75 60.75 C160 81, 160 81, 139.75 101.25 M139.75 60.75 C160 81, 160 81, 139.75 101.25 M139.75 101.25 C125.55 115.45, 111.35 129.65, 101.25 139.75 M139.75 101.25 C124.35 116.65, 108.95 132.05, 101.25 139.75 M101.25 139.75 C81 160, 81 160, 60.75 139.75 M101.25 139.75 C81 160, 81 160, 60.75 139.75 M60.75 139.75 C51.04 130.52, 41.32 121.28, 20.25 101.25 M60.75 139.75 C48.21 127.83, 35.67 115.91, 20.25 101.25 M20.25 101.25 C0 81, 0 81, 20.25 60.75 M20.25 101.25 C0 81, 0 81, 20.25 60.75 M20.25 60.75 C34.48 46.52, 48.72 32.28, 60.75 20.25 M20.25 60.75 C33.39 47.61, 46.52 34.48, 60.75 20.25 M60.75 20.25 C81 0, 81 0, 101.25 20.25 M60.75 20.25 C81 0, 81 0, 101.25 20.25" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(58 2870) rotate(0 32 20)"><text x="32" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">缓存何时</text><text x="32" y="34" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">失效?</text></g><g stroke-linecap="round" transform="translate(10 3130) rotate(0 80 81.5)"><path d="M32 0 C65.59 0, 99.17 0, 128 0 M32 0 C66.04 0, 100.08 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 61.12, 160 90.23, 160 131 M160 32 C160 68.95, 160 105.9, 160 131 M160 131 C160 152.33, 149.33 163, 128 163 M160 131 C160 152.33, 149.33 163, 128 163 M128 163 C103.41 163, 78.82 163, 32 163 M128 163 C102.58 163, 77.15 163, 32 163 M32 163 C10.67 163, 0 152.33, 0 131 M32 163 C10.67 163, 0 152.33, 0 131 M0 131 C0 96.96, 0 62.93, 0 32 M0 131 C0 97.5, 0 64.01, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(26 3201.5) rotate(0 64 10)"><text x="64" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">缓存相应条目失效</text></g><g stroke-linecap="round" transform="translate(330 3130) rotate(0 80 81.5)"><path d="M32 0 C59.26 0, 86.52 0, 128 0 M32 0 C54.01 0, 76.01 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 60.48, 160 88.95, 160 131 M160 32 C160 70.01, 160 108.02, 160 131 M160 131 C160 152.33, 149.33 163, 128 163 M160 131 C160 152.33, 149.33 163, 128 163 M128 163 C107.01 163, 86.03 163, 32 163 M128 163 C106.55 163, 85.1 163, 32 163 M32 163 C10.67 163, 0 152.33, 0 131 M32 163 C10.67 163, 0 152.33, 0 131 M0 131 C0 109.14, 0 87.27, 0 32 M0 131 C0 92.14, 0 53.29, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(362 3201.5) rotate(0 48 10)"><text x="48" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">清空所有缓存</text></g><g stroke-linecap="round" transform="translate(10 3450) rotate(0 80 81.5)"><path d="M32 0 C52.04 0, 72.08 0, 128 0 M32 0 C62.1 0, 92.21 0, 128 0 M128 0 C149.33 0, 160 10.67, 160 32 M128 0 C149.33 0, 160 10.67, 160 32 M160 32 C160 61.95, 160 91.89, 160 131 M160 32 C160 59.11, 160 86.23, 160 131 M160 131 C160 152.33, 149.33 163, 128 163 M160 131 C160 152.33, 149.33 163, 128 163 M128 163 C96.22 163, 64.43 163, 32 163 M128 163 C93.05 163, 58.1 163, 32 163 M32 163 C10.67 163, 0 152.33, 0 131 M32 163 C10.67 163, 0 152.33, 0 131 M0 131 C0 95.15, 0 59.3, 0 32 M0 131 C0 103.57, 0 76.13, 0 32 M0 32 C0 10.67, 10.67 0, 32 0 M0 32 C0 10.67, 10.67 0, 32 0" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(16.4000244140625 3511.5) rotate(0 73.5999755859375 20)"><text x="73.5999755859375" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">文件再次被请求时</text><text x="73.5999755859375" y="34" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">&lt;br&gt;重新编译并缓存</text></g><g stroke-linecap="round"><g transform="translate(89.90000000000009 2655) rotate(0 0.013923156176019802 75.79801396686923)"><path d="M0 0 C0.01 35.04, 0.01 70.08, 0.03 151.6 M0 0 C0.01 35.34, 0.01 70.69, 0.03 151.6" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.90000000000009 2655) rotate(0 0.013923156176019802 75.79801396686923)"><path d="M0.03 151.6 L-6.31 138 L6.36 138 L0.03 151.6" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0.03 151.6 C-1.44 148.45, -2.9 145.31, -6.31 138 M0.03 151.6 C-1.45 148.43, -2.93 145.26, -6.31 138 M-6.31 138 C-2.67 138, 0.98 138, 6.36 138 M-6.31 138 C-1.45 138, 3.42 138, 6.36 138 M6.36 138 C4.84 141.26, 3.32 144.52, 0.03 151.6 M6.36 138 C4.4 142.21, 2.44 146.42, 0.03 151.6 M0.03 151.6 C0.03 151.6, 0.03 151.6, 0.03 151.6 M0.03 151.6 C0.03 151.6, 0.03 151.6, 0.03 151.6" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g mask="url(#mask-9JP36OnpHn_-dqX6dGG-c)" stroke-linecap="round"><g transform="translate(89.92784631235213 2973.4039720662613) rotate(0 -0.013923156176019802 75.79801396686923)"><path d="M0 0 C-0.01 43.73, -0.02 87.45, -0.03 151.6 M0 0 C-0.01 32.97, -0.01 65.94, -0.03 151.6" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.92784631235213 2973.4039720662613) rotate(0 -0.013923156176019802 75.79801396686923)"><path d="M-0.03 151.6 L-6.36 138 L6.31 138 L-0.03 151.6" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M-0.03 151.6 C-1.86 147.67, -3.68 143.75, -6.36 138 M-0.03 151.6 C-1.41 148.64, -2.78 145.68, -6.36 138 M-6.36 138 C-3.51 138, -0.66 138, 6.31 138 M-6.36 138 C-1.82 138, 2.72 138, 6.31 138 M6.31 138 C4.72 141.41, 3.13 144.82, -0.03 151.6 M6.31 138 C4.62 141.64, 2.92 145.28, -0.03 151.6 M-0.03 151.6 C-0.03 151.6, -0.03 151.6, -0.03 151.6 M-0.03 151.6 C-0.03 151.6, -0.03 151.6, -0.03 151.6" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask id="mask-9JP36OnpHn_-dqX6dGG-c" maskUnits="userSpaceOnUse" x="0" y="0" width="189.95569262470417" height="3225"><rect x="0" y="0" fill="#fff" width="189.95569262470417" height="3225"/><rect x="28.913923156176224" y="3034.20198603313" fill="#000" width="122" height="30" opacity="1"/></mask><g transform="translate(33.913923156176224 3039.2019860331307) rotate(0 55.999999999999886 10.000000000000114)"><text x="56" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">文件更新或过期</text></g><g mask="url(#mask-nRd3IgM5YcZ2Q3WeFFE7C)" stroke-linecap="round"><g transform="translate(169.889553337591 2889.900820296275) rotate(0 120.00522333120455 117.5495898518626)"><path d="M0 0 C82.62 0, 165.24 0, 224.01 0 M0 0 C89.59 0, 179.19 0, 224.01 0 M224.01 0 C234.68 0, 240.01 5.33, 240.01 16 M224.01 0 C234.68 0, 240.01 5.33, 240.01 16 M240.01 16 C240.01 68.55, 240.01 121.11, 240.01 235.1 M240.01 16 C240.01 83.84, 240.01 151.68, 240.01 235.1" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(169.889553337591 2889.900820296275) rotate(0 120.00522333120455 117.5495898518626)"><path d="M240.01 235.1 L233.67 221.5 L246.35 221.5 L240.01 235.1" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M240.01 235.1 C237.67 230.09, 235.33 225.07, 233.67 221.5 M240.01 235.1 C237.48 229.66, 234.94 224.22, 233.67 221.5 M233.67 221.5 C237.84 221.5, 242.01 221.5, 246.35 221.5 M233.67 221.5 C236.48 221.5, 239.29 221.5, 246.35 221.5 M246.35 221.5 C243.97 226.6, 241.59 231.7, 240.01 235.1 M246.35 221.5 C244.45 225.58, 242.55 229.65, 240.01 235.1 M240.01 235.1 C240.01 235.1, 240.01 235.1, 240.01 235.1 M240.01 235.1 C240.01 235.1, 240.01 235.1, 240.01 235.1" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask id="mask-nRd3IgM5YcZ2Q3WeFFE7C" maskUnits="userSpaceOnUse" x="0" y="0" width="509.9000000000001" height="3225"><rect x="0" y="0" fill="#fff" width="509.9000000000001" height="3225"/><rect x="356.9000000000001" y="2874.900820296275" fill="#000" width="106" height="30" opacity="1"/></mask><g transform="translate(361.9000000000001 2879.900820296275) rotate(0 -72.00522333120455 127.5495898518626)"><text x="48" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">手动重置缓存</text></g><g stroke-linecap="round"><g transform="translate(398.1269765389409 3298) rotate(0 -154.1134882694704 73.5)"><path d="M0 0 C0 14.31, 0 28.62, 0 57.5 M0 0 C0 13.64, 0 27.28, 0 57.5 M0 57.5 C0 68.17, -5.33 73.5, -16 73.5 M0 57.5 C0 68.17, -5.33 73.5, -16 73.5 M-16 73.5 C-119.6 73.5, -223.19 73.5, -292.23 73.5 M-16 73.5 C-98.76 73.5, -181.52 73.5, -292.23 73.5 M-292.23 73.5 C-302.89 73.5, -308.23 78.83, -308.23 89.5 M-292.23 73.5 C-302.89 73.5, -308.23 78.83, -308.23 89.5 M-308.23 89.5 C-308.23 109.44, -308.23 129.38, -308.23 147 M-308.23 89.5 C-308.23 106.99, -308.23 124.49, -308.23 147" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(398.1269765389409 3298) rotate(0 -154.1134882694704 73.5)"><path d="M-308.23 147 L-314.57 133.41 L-301.89 133.41 L-308.23 147" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M-308.23 147 C-309.8 143.62, -311.38 140.23, -314.57 133.41 M-308.23 147 C-309.73 143.78, -311.23 140.55, -314.57 133.41 M-314.57 133.41 C-311.31 133.41, -308.06 133.41, -301.89 133.41 M-314.57 133.41 C-311.58 133.41, -308.59 133.41, -301.89 133.41 M-301.89 133.41 C-303.99 137.91, -306.09 142.41, -308.23 147 M-301.89 133.41 C-304.2 138.35, -306.5 143.3, -308.23 147 M-308.23 147 C-308.23 147, -308.23 147, -308.23 147 M-308.23 147 C-308.23 147, -308.23 147, -308.23 147" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(89.90000000000009 2335) rotate(0 0.0015151515151501371 69.07894736842104)"><path d="M0 0 C0 40.17, 0 80.34, 0 138.16 M0 0 C0 44.47, 0 88.93, 0 138.16" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.90000000000009 2335) rotate(0 0.0015151515151501371 69.07894736842104)"><path d="M0 138.16 L-6.34 124.56 L6.34 124.56 L0 138.16" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 138.16 C-1.84 134.21, -3.68 130.25, -6.34 124.56 M0 138.16 C-2.04 133.78, -4.08 129.41, -6.34 124.56 M-6.34 124.56 C-2.79 124.56, 0.76 124.56, 6.34 124.56 M-6.34 124.56 C-3.65 124.56, -0.97 124.56, 6.34 124.56 M6.34 124.56 C4.5 128.51, 2.66 132.45, 0 138.16 M6.34 124.56 C4.48 128.56, 2.62 132.55, 0 138.16 M0 138.16 C0 138.16, 0 138.16, 0 138.16 M0 138.16 C0 138.16, 0 138.16, 0 138.16" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round" transform="translate(10 10) rotate(0 80 40)"><path d="M160 40 C160 41.61, 159.8 43.23, 159.42 44.82 C159.03 46.42, 158.44 48.01, 157.68 49.57 C156.91 51.13, 155.94 52.68, 154.8 54.18 C153.66 55.69, 152.33 57.17, 150.84 58.59 C149.34 60.01, 147.66 61.4, 145.84 62.72 C144.01 64.05, 142.01 65.32, 139.88 66.52 C137.75 67.73, 135.46 68.87, 133.05 69.94 C130.64 71.01, 128.09 72.01, 125.45 72.92 C122.8 73.83, 120.02 74.67, 117.18 75.42 C114.33 76.17, 111.37 76.83, 108.37 77.4 C105.36 77.97, 102.27 78.45, 99.15 78.84 C96.02 79.22, 92.83 79.51, 89.64 79.71 C86.45 79.9, 83.21 80, 80 80 C76.79 80, 73.55 79.9, 70.36 79.71 C67.17 79.51, 63.98 79.22, 60.85 78.84 C57.73 78.45, 54.64 77.97, 51.63 77.4 C48.63 76.83, 45.67 76.17, 42.82 75.42 C39.98 74.67, 37.2 73.83, 34.55 72.92 C31.91 72.01, 29.36 71.01, 26.95 69.94 C24.54 68.87, 22.25 67.73, 20.12 66.52 C17.99 65.32, 15.99 64.05, 14.16 62.72 C12.34 61.4, 10.66 60.01, 9.16 58.59 C7.67 57.17, 6.34 55.69, 5.2 54.18 C4.06 52.68, 3.09 51.13, 2.32 49.57 C1.56 48.01, 0.97 46.42, 0.58 44.82 C0.2 43.23, 0 41.61, 0 40 C0 38.39, 0.2 36.77, 0.58 35.18 C0.97 33.58, 1.56 31.99, 2.32 30.43 C3.09 28.87, 4.06 27.32, 5.2 25.82 C6.34 24.31, 7.67 22.83, 9.16 21.41 C10.66 19.99, 12.34 18.6, 14.16 17.28 C15.99 15.95, 17.99 14.68, 20.12 13.48 C22.25 12.27, 24.54 11.13, 26.95 10.06 C29.36 8.99, 31.91 7.99, 34.55 7.08 C37.2 6.17, 39.98 5.33, 42.82 4.58 C45.67 3.83, 48.63 3.17, 51.63 2.6 C54.64 2.03, 57.73 1.55, 60.85 1.16 C63.98 0.78, 67.17 0.49, 70.36 0.29 C73.55 0.1, 76.79 0, 80 0 C83.21 0, 86.45 0.1, 89.64 0.29 C92.83 0.49, 96.02 0.78, 99.15 1.16 C102.27 1.55, 105.36 2.03, 108.37 2.6 C111.37 3.17, 114.33 3.83, 117.18 4.58 C120.02 5.33, 122.8 6.17, 125.45 7.08 C128.09 7.99, 130.64 8.99, 133.05 10.06 C135.46 11.13, 137.75 12.27, 139.88 13.48 C142.01 14.68, 144.01 15.95, 145.84 17.28 C147.66 18.6, 149.34 19.99, 150.84 21.41 C152.33 22.83, 153.66 24.31, 154.8 25.82 C155.94 27.32, 156.91 28.87, 157.68 30.43 C158.44 31.99, 159.03 33.58, 159.42 35.18 C159.8 36.77, 160 38.39, 160 40" stroke="#1e1e1e" stroke-width="1" fill="none"/></g><g transform="translate(73.9314575050762 40.2157287525381) rotate(0 16 10)"><text x="16" y="15.264000000000001" font-family="Nunito, sans-serif, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">开始</text></g><g stroke-linecap="round" transform="translate(10 3690) rotate(0 80 40)"><path d="M160 40 C160 41.61, 159.8 43.23, 159.42 44.82 C159.03 46.42, 158.44 48.01, 157.68 49.57 C156.91 51.13, 155.94 52.68, 154.8 54.18 C153.66 55.69, 152.33 57.17, 150.84 58.59 C149.34 60.01, 147.66 61.4, 145.84 62.72 C144.01 64.05, 142.01 65.32, 139.88 66.52 C137.75 67.73, 135.46 68.87, 133.05 69.94 C130.64 71.01, 128.09 72.01, 125.45 72.92 C122.8 73.83, 120.02 74.67, 117.18 75.42 C114.33 76.17, 111.37 76.83, 108.37 77.4 C105.36 77.97, 102.27 78.45, 99.15 78.84 C96.02 79.22, 92.83 79.51, 89.64 79.71 C86.45 79.9, 83.21 80, 80 80 C76.79 80, 73.55 79.9, 70.36 79.71 C67.17 79.51, 63.98 79.22, 60.85 78.84 C57.73 78.45, 54.64 77.97, 51.63 77.4 C48.63 76.83, 45.67 76.17, 42.82 75.42 C39.98 74.67, 37.2 73.83, 34.55 72.92 C31.91 72.01, 29.36 71.01, 26.95 69.94 C24.54 68.87, 22.25 67.73, 20.12 66.52 C17.99 65.32, 15.99 64.05, 14.16 62.72 C12.34 61.4, 10.66 60.01, 9.16 58.59 C7.67 57.17, 6.34 55.69, 5.2 54.18 C4.06 52.68, 3.09 51.13, 2.32 49.57 C1.56 48.01, 0.97 46.42, 0.58 44.82 C0.2 43.23, 0 41.61, 0 40 C0 38.39, 0.2 36.77, 0.58 35.18 C0.97 33.58, 1.56 31.99, 2.32 30.43 C3.09 28.87, 4.06 27.32, 5.2 25.82 C6.34 24.31, 7.67 22.83, 9.16 21.41 C10.66 19.99, 12.34 18.6, 14.16 17.28 C15.99 15.95, 17.99 14.68, 20.12 13.48 C22.25 12.27, 24.54 11.13, 26.95 10.06 C29.36 8.99, 31.91 7.99, 34.55 7.08 C37.2 6.17, 39.98 5.33, 42.82 4.58 C45.67 3.83, 48.63 3.17, 51.63 2.6 C54.64 2.03, 57.73 1.55, 60.85 1.16 C63.98 0.78, 67.17 0.49, 70.36 0.29 C73.55 0.1, 76.79 0, 80 0 C83.21 0, 86.45 0.1, 89.64 0.29 C92.83 0.49, 96.02 0.78, 99.15 1.16 C102.27 1.55, 105.36 2.03, 108.37 2.6 C111.37 3.17, 114.33 3.83, 117.18 4.58 C120.02 5.33, 122.8 6.17, 125.45 7.08 C128.09 7.99, 130.64 8.99, 133.05 10.06 C135.46 11.13, 137.75 12.27, 139.88 13.48 C142.01 14.68, 144.01 15.95, 145.84 17.28 C147.66 18.6, 149.34 19.99, 150.84 21.41 C152.33 22.83, 153.66 24.31, 154.8 25.82 C155.94 27.32, 156.91 28.87, 157.68 30.43 C158.44 31.99, 159.03 33.58, 159.42 35.18 C159.8 36.77, 160 38.39, 160 40" stroke="#1e1e1e" stroke-width="1" fill="none"/></g><g transform="translate(73.9314575050762 3720.215728752538) rotate(0 16 10)"><text x="16" y="15.264000000000001" font-family="Nunito, sans-serif, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">结束</text></g><g stroke-linecap="round"><g transform="translate(89.89999999999986 3298) rotate(0 1.1368683772161603e-13 73.5)"><path d="M0 0 C0 58.57, 0 117.13, 0 147 M0 0 C0 36.14, 0 72.28, 0 147" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.89999999999986 3298) rotate(0 1.1368683772161603e-13 73.5)"><path d="M0 147 L-6.34 133.41 L6.34 133.41 L0 147" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 147 C-2.53 141.58, -5.05 136.17, -6.34 133.41 M0 147 C-1.56 143.66, -3.12 140.32, -6.34 133.41 M-6.34 133.41 C-3.09 133.41, 0.16 133.41, 6.34 133.41 M-6.34 133.41 C-1.58 133.41, 3.19 133.41, 6.34 133.41 M6.34 133.41 C3.88 138.69, 1.41 143.97, 0 147 M6.34 133.41 C4.77 136.76, 3.21 140.12, 0 147 M0 147 C0 147, 0 147, 0 147 M0 147 C0 147, 0 147, 0 147" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round" transform="translate(10 890) rotate(0 80 80.62015503875966)"><path d="M101.25 20.25 C109.71 29.15, 118.18 38.06, 139.75 60.75 M101.25 20.25 C114.77 34.47, 128.28 48.69, 139.75 60.75 M139.75 60.75 C160 81, 160 81, 139.75 101.25 M139.75 60.75 C160 81, 160 81, 139.75 101.25 M139.75 101.25 C127 114.41, 114.25 127.58, 101.25 140.99 M139.75 101.25 C131.05 110.23, 122.35 119.22, 101.25 140.99 M101.25 140.99 C81 161.24, 81 161.24, 60.75 140.99 M101.25 140.99 C81 161.24, 81 161.24, 60.75 140.99 M60.75 140.99 C52.28 132.68, 43.8 124.36, 20.25 101.25 M60.75 140.99 C46.02 126.54, 31.3 112.09, 20.25 101.25 M20.25 101.25 C0 81, 0 81, 20.25 60.75 M20.25 101.25 C0 81, 0 81, 20.25 60.75 M20.25 60.75 C30.76 50.24, 41.27 39.73, 60.75 20.25 M20.25 60.75 C32.9 48.1, 45.55 35.45, 60.75 20.25 M60.75 20.25 C81 0, 81 0, 101.25 20.25 M60.75 20.25 C81 0, 81 0, 101.25 20.25" stroke="#1e1e1e" stroke-width="1" fill="none"/></g><g transform="translate(69.60000610351562 950.8100775193798) rotate(0 20.399993896484375 20)"><text x="20.399993896484375" y="14" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">缓存</text><text x="20.399993896484375" y="34" font-family="Comic Shanns, monospace, Segoe UI Emoji" font-size="16px" fill="#1e1e1e" text-anchor="middle" style="white-space: pre;" direction="ltr" dominant-baseline="alphabetic">命中?</text></g><g stroke-linecap="round"><g transform="translate(89.90000000000009 743.3333333333333) rotate(0 0.0004101481374618743 73.38855666453787)"><path d="M0 0 C0 41.95, 0 83.91, 0 146.78 M0 0 C0 43.12, 0 86.24, 0 146.78" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.90000000000009 743.3333333333333) rotate(0 0.0004101481374618743 73.38855666453787)"><path d="M0 146.78 L-6.34 133.18 L6.34 133.18 L0 146.78" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 146.78 C-1.81 142.89, -3.62 139.01, -6.34 133.18 M0 146.78 C-1.86 142.78, -3.72 138.79, -6.34 133.18 M-6.34 133.18 C-2.84 133.18, 0.67 133.18, 6.34 133.18 M-6.34 133.18 C-3.51 133.18, -0.69 133.18, 6.34 133.18 M6.34 133.18 C5.01 136.03, 3.68 138.89, 0 146.78 M6.34 133.18 C4.02 138.15, 1.7 143.12, 0 146.78 M0 146.78 C0 146.78, 0 146.78, 0 146.78 M0 146.78 C0 146.78, 0 146.78, 0 146.78" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(89.90082029627501 1051.1298634151103) rotate(0 -0.0004101481374618743 390.6192788187606)"><path d="M0 0 C0 173.65, 0 347.3, 0 781.24 M0 0 C0 295.25, 0 590.5, 0 781.24" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(89.90082029627501 1051.1298634151103) rotate(0 -0.0004101481374618743 390.6192788187606)"><path d="M0 781.24 L-6.34 767.64 L6.34 767.64 L0 781.24" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 781.24 C-1.41 778.22, -2.82 775.2, -6.34 767.64 M0 781.24 C-2.4 776.1, -4.79 770.96, -6.34 767.64 M-6.34 767.64 C-2.72 767.64, 0.91 767.64, 6.34 767.64 M-6.34 767.64 C-3.63 767.64, -0.92 767.64, 6.34 767.64 M6.34 767.64 C5.06 770.39, 3.78 773.13, 0 781.24 M6.34 767.64 C5.06 770.38, 3.79 773.12, 0 781.24 M0 781.24 C0 781.24, 0 781.24, 0 781.24 M0 781.24 C0 781.24, 0 781.24, 0 781.24" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(409.9000088715452 1214.394672261136) rotate(0 -0.000004435772439137509 75.30266386943197)"><path d="M0 0 C0 54.16, 0 108.32, 0 150.61 M0 0 C0 36.67, 0 73.34, 0 150.61" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(409.9000088715452 1214.394672261136) rotate(0 -0.000004435772439137509 75.30266386943197)"><path d="M0 150.61 L-6.34 137.01 L6.34 137.01 L0 150.61" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M0 150.61 C-2.28 145.72, -4.56 140.83, -6.34 137.01 M0 150.61 C-1.54 147.3, -3.09 143.99, -6.34 137.01 M-6.34 137.01 C-2.38 137.01, 1.57 137.01, 6.34 137.01 M-6.34 137.01 C-2.35 137.01, 1.64 137.01, 6.34 137.01 M6.34 137.01 C4.7 140.53, 3.06 144.05, 0 150.61 M6.34 137.01 C5.02 139.84, 3.7 142.67, 0 150.61 M0 150.61 C0 150.61, 0 150.61, 0 150.61 M0 150.61 C0 150.61, 0 150.61, 0 150.61" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/><g stroke-linecap="round"><g transform="translate(729.9000000000001 1855) rotate(0 -117.45000000000005 197.45)"><path d="M0 0 C0 125.52, 0 251.04, 0 378.9 M0 0 C0 118.71, 0 237.41, 0 378.9 M0 378.9 C0 389.57, -5.33 394.9, -16 394.9 M0 378.9 C0 389.57, -5.33 394.9, -16 394.9 M-16 394.9 C-66.82 394.9, -117.64 394.9, -234.9 394.9 M-16 394.9 C-83.81 394.9, -151.63 394.9, -234.9 394.9" stroke="#1e1e1e" stroke-width="2" fill="none"/></g><g transform="translate(729.9000000000001 1855) rotate(0 -117.45000000000005 197.45)"><path d="M-234.9 394.9 L-221.31 388.56 L-221.31 401.24 L-234.9 394.9" stroke="none" stroke-width="0" fill="#1e1e1e" fill-rule="evenodd"/><path d="M-234.9 394.9 C-230.4 392.8, -225.89 390.7, -221.31 388.56 M-234.9 394.9 C-230.64 392.91, -226.38 390.93, -221.31 388.56 M-221.31 388.56 C-221.31 391.48, -221.31 394.4, -221.31 401.24 M-221.31 388.56 C-221.31 392.15, -221.31 395.74, -221.31 401.24 M-221.31 401.24 C-224.23 399.87, -227.16 398.51, -234.9 394.9 M-221.31 401.24 C-224.11 399.93, -226.92 398.62, -234.9 394.9 M-234.9 394.9 C-234.9 394.9, -234.9 394.9, -234.9 394.9 M-234.9 394.9 C-234.9 394.9, -234.9 394.9, -234.9 394.9" stroke="#1e1e1e" stroke-width="2" fill="none"/></g></g><mask/></svg></div>
# 三、核心机制  
## 1. 字节码缓存内容  
### OPcache 缓存的内容
#### 预编译的字节码（Opcode）
- **定义**：PHP 脚本被解析后生成的中间代码（操作码），可以直接由 Zend 引擎执行。
- **举例**：
    - WordPress 的核心文件（如 `wp-blog-header.php`）的编译结果。
    - 框架（如 Laravel）中控制器、模型的字节码。
#### 函数和类定义
- **定义**：PHP 脚本中定义的函数、类及其方法的结构信息。
- **举例**：
```php
class MyClass {
  public function sayHello() {
	  return "Hello";
  }
}
// 编译后，MyClass的Opcode被缓存。
```  
#### 文件路径和依赖关系
- **定义**：PHP 文件的路径、修改时间（MTIME）以及文件包含关系（如 `include`、`require`）。
- **举例**：
    - 文件 `functions.php` 的路径 `/var/www/html/wp-content/themes/mytheme/functions.php`。
    - 包含关系 `require 'config.php'` 的记录。
```php
require 'config.php'; // 缓存路径及config.php的mtime。
```  
- **Interned Strings**：优化字符串存储（如变量名、函数名），减少内存占用。  
#### Interned Strings（内部化字符串）
- **定义**：频繁使用的字符串（如变量名、类名、方法名）的内存优化存储，避免重复分配内存。
- **举例**：
    - 变量名 `$user_id`、类名 `WP_Post`、方法名 `__construct`。
    - 字符串常量 `"admin"` 或 `"public"`。
#### OPArray 结构
- **定义**：PHP 脚本的抽象语法树（AST）编译后的执行结构，包含控制流和操作指令。
- **举例**：
    - 循环 `for ($i=0; $i<10; $i++)` 的执行逻辑。
    - 条件语句 `if (is_user_logged_in())` 的分支结构。
### OPcache 不缓存的内容
OPcache 的设计仅针对 PHP 脚本的编译结果，以下内容不在其缓存范围内：
#### 运行时数据
- **定义**：程序执行过程中动态生成的数据，如变量值、会话信息、数据库查询结果。
- **举例**：
    - 用户登录后的 `$user->name = 'John'`。
    - 数据库查询结果 `SELECT * FROM posts` 的记录集。
#### 未被包含的文件
- **定义**：未通过 `include`、`require` 等语句加载的 PHP 文件。
- **举例**：
    - 未被调用的插件文件 `unused-plugin.php`。
    - 未被包含的测试文件 `tests/unit_test.php`。
#### 动态生成的代码（如 eval() 的结果）
- **定义**：通过 `eval()` 或 `create_function()` 动态执行的代码。
- **举例**：
```php
eval("echo 'Hello World';"); // 动态生成的代码不会被缓存。
```
#### 注释（可配置）
- **定义**：PHP 文件中的注释默认会被缓存，但可以通过配置 `opcache.save_comments=0` 禁用。
- **举例**：
    - 文档注释 `/** @param int $id */`。
    - 普通注释 `// 这是一个测试`。
## 2. 共享内存管理  
- **共享内存类型**：  
  - `mmap`（默认）：非持久化，进程退出后释放。  
  - `System V shm`：持久化，需手动清理。  
- **内存分配**：通过`opcache.memory_consumption`配置总内存（如256MB），内存不足时按LRU算法淘汰旧缓存。  
## 3. 互斥锁（Mutex）机制  
- **并发控制**：  
  - 读操作允许多进程并发，无需锁。  
  - 写操作仅一个进程可执行，其他进程等待锁释放。  
- **锁竞争**：高并发下，频繁修改脚本可能导致进程排队，需合理设置`opcache.revalidate_freq`（默认2秒）。  
## 4. 缓存更新与失效  
- **自动检测**：通过文件时间戳变化触发重新编译。  
- **强制清除**：使用`opcache_invalidate('file.php', true)`或`opcache_reset()`。  
# 四、性能提升点  
1. **减少解析和编译开销**：直接执行缓存的字节码，降低CPU和内存消耗。  
2. **代码优化**：消除无用操作（如重复计算），合并重复代码。  
3. **共享内存复用**：多PHP-FPM进程共享字节码，降低内存占用。  
**示例**：  
- **WordPress网站**：缓存所有PHP文件的Opcode后，响应时间从500ms降至50ms。  
- **Laravel微服务API**：通过OPcache缓存`vendor`目录的类文件，减少90%的编译时间。  
# 五、配置与调优  
## 1. 关键参数  
| **参数**                          | **作用**             | **建议值**          |     |
| ------------------------------- | ------------------ | ---------------- | --- |
| `opcache.enable`                | 启用OPcache（1启用，0禁用） | 生产环境设为1          |     |
| `opcache.memory_consumption`    | 共享内存总大小（MB）        | 根据内存调整，如256或更高   |     |
| `opcache.max_accelerated_files` | 允许缓存的最大文件数         | Laravel项目设为20000 |     |
| `opcache.revalidate_freq`       | 文件修改检查频率（秒）        | 生产环境设为60，开发环境设为0 |     |
| `opcache.validate_timestamps`   | 是否检查文件时间戳（1启用，0禁用） | 生产环境设为0以避免频繁检查   |     |
## 2. 注意事项  
- **与Xdebug的兼容性**：Xdebug的调试功能会禁用OPcache优化，生产环境需禁用Xdebug。  
- **开发环境配置**：
```ini
[opcache]
opcache.enable=0
opcache.validate_timestamps=1
```  
# 六、适用场景与限制  
## 1. 适用场景  
- **加速PHP执行**：如WordPress、Laravel框架的PHP文件编译。  
- **代码优化**：自动优化字节码以提升执行速度。  
## 2. 重要限制  
- **不缓存文件内容**：OPcache仅缓存PHP脚本的字节码，不会缓存通过`file_get_contents()`加载的文件内容。  
  **解决方案**：  
  - 使用Redis/Memcached缓存文件内容。  
  - 对PHP配置文件，通过`include`引入，利用OPcache缓存其字节码。  
# 七、与其他技术的对比  
| **类型**   | **OPcache**  | **文件内容缓存（如Redis）** |
| -------- | ------------ | ------------------ |
| **缓存内容** | PHP字节码       | 文件内容（JSON、配置文件等）   |
| **核心作用** | 减少PHP解析/编译开销 | 减少磁盘I/O，加速文件读取     |
| **配置方式** | 内置PHP配置参数    | 需手动编码实现缓存逻辑        |
| **适用场景** | PHP脚本执行加速    | 频繁读取的静态文件或配置       |
# 八、实际应用与验证  
## 1. 验证OPcache效果  
### 1. 启用OPcache  
在`php.ini`中设置`opcache.enable=1`，重启PHP-FPM。
### 2. 查看 `OPcache` 配置配置信息
```php
php -i | grep opcache
```
### 3. 监控状态
#### 脚本监控
通过`opcache_get_status()`查看关键指标
```php
$opcache_status = opcache_get_status();
echo json_encode($opcache_status);
```
>理想状态：命中率应超过 90%，否则需调整配置（如增加 `memory_consumption` 或 `max_accelerated_files`）。
#### 计算 max_accelerated_files 的数量
OPcache 缓存的最大文件数量 (根据应用调整，确保足够大以容纳所有 PHP 文件)
```shell
find . -type f -name "*.php" | wc -l
```
#### 图形化监控「OPcache GUI」
##### 安装
```php
# 克隆代码
git clone git@github.com:PeeHaa/OpCacheGUI.git
# 切换到根目录并拷贝配置文件
cd OpCacheGUI && cp config.sample.php config.php
# 添加nginx配置
server {
    listen       90;
    server_name  _;
    root   /www/localhost/OpCacheGUI/public;
    index  index.php index.html index.htm;

    location / {
          try_files $uri $uri/ /index.php?$query_string;
    }
    access_log /dev/null;
    error_log  /var/log/nginx/opcache.gui2.error.log  warn;
    location ~ \.php$ {
          fastcgi_pass   php80:9000;
          fastcgi_index  index.php;
          include        fastcgi_params;
          fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    }
}
```
##### 使用步骤：
1. 访问 `http://10.0.0.6:90`，查看缓存命中率、内存使用、已缓存文件列表等。
```json
{
    "opcache_enabled": true,            // OPcache 是否启用 (true = 启用, false = 禁用)
    "cache_full": false,                // OPcache 缓存是否已满 (true = 已满, false = 未满)
    "restart_pending": false,           // 是否有重启请求等待执行 (true = 等待中, false = 否)
    "restart_in_progress": false,      // 是否正在进行重启 (true = 进行中, false = 否)
    "memory_usage": {
        "used_memory": 23648032,        // 已使用的内存 (字节)
        "free_memory": 110563008,       // 可用内存 (字节)
        "wasted_memory": 6688,          // 浪费的内存 (字节)
        "current_wasted_percentage": 0.004982948303222656 // 当前内存浪费百分比
    },
    "interned_strings_usage": {
        "buffer_size": 6290992,        // 内部字符串缓冲区大小 (字节)
        "used_memory": 3930744,        // 已使用的内部字符串内存 (字节)
        "free_memory": 2360248,        // 可用的内部字符串内存 (字节)
        "number_of_strings": 57142     // 内部字符串数量
    },
    "opcache_statistics": {
        "num_cached_scripts": 844,      // 已缓存的脚本数量
        "num_cached_keys": 1608,        // 已缓存的键数量
        "max_cached_keys": 16229,       // 最大缓存键数量
        "hits": 423611,                // 缓存命中次数
        "start_time": 1744158638,       // OPcache 启动时间 (Unix 时间戳)
        "last_restart_time": 0,         // 上次重启时间 (Unix 时间戳, 0 表示未重启)
        "oom_restarts": 0,              // 内存不足重启次数
        "hash_restarts": 0,             // 哈希冲突重启次数
        "manual_restarts": 0,           // 手动重启次数
        "misses": 855,                  // 缓存未命中次数
        "blacklist_misses": 0,          // 黑名单未命中次数
        "blacklist_miss_ratio": 0,      // 黑名单未命中率
        "opcache_hit_rate": 99.79857043909288 // OPcache 命中率 (%)
    }
}
```
## 2. 典型配置示例（生产环境）  
```ini
[opcache]
; 启用 OPcache 扩展 (必须启用) (1 = 启用, 0 = 禁用)
opcache.enable=1
; CLI 模式也启用 Opcache (推荐)
opcache.enable_cli=1
; OPcache 使用的共享内存大小，单位为 MB (根据应用调整，建议 256MB 起步)
opcache.memory_consumption=256
; OPcache 缓存的最大文件数量 (根据应用调整，确保足够大以容纳所有 PHP 文件)
opcache.max_accelerated_files=20000
; 原理：当 validate_timestamps=1 时生效，定义检查文件更新的时间间隔（秒）。
; =0：每个请求都检查文件更新（性能最差，仅适合开发环境）。
; =N（如 60）：每 N 秒检查一次，期间忽略文件修改
opcache.revalidate_freq=2
; 原理：控制 OPcache 是否检查 PHP 文件的修改时间（mtime）。
; =1（启用）：定期检查文件是否更新，若更新则重新编译缓存。
; =0（禁用）：完全跳过文件更新检查，缓存永不自动失效（需手动重置）
opcache.validate_timestamps=0
; 用于存储内部字符串的内存大小「避免存储重复的字符到内存引起的浪费」，单位为 MB。(根据应用调整，如果应用大量重复字符串操作，可以增加此值)
; interned strings 机制会创建一个特殊的共享内存池（Interned Strings Buffer）。当 PHP 遇到一个字符串时，它会首先检查这个字符串是否已经在内存池中。
; 如果存在，它会直接创建一个指向该字符串在内存池中地址的引用。
; 如果不存在，它会将这个字符串添加到内存池中，然后创建一个引用。
opcache.interned_strings_buffer=32
; 优化内存分配 (推荐启用)
opcache.optimization_level=0x7FFFBFFF
; 禁用文件统计，提高性能 (推荐启用)
opcache.use_cwd=1
; opcache.error_log=/var/log/php-fpm-opcache.log
; 含义：指定 Opcache 扩展自身的错误日志文件路径。
; 推荐：生产环境推荐开启并设置一个绝对路径。
; 理由：Opcache 扩展在运行时可能会遇到错误或警告（例如内存不足、文件无法缓存等）。
;       将这些错误记录到单独的日志文件，有助于监控 Opcache 的健康状况，及时发现并解决问题，
;       确保 Opcache 正常工作，从而发挥其性能优化作用。
;       确保日志文件路径存在，且 PHP-FPM 运行用户（如 www-data）对该路径有写入权限。
opcache.error_log=/var/log/php-fpm-opcache.log

; opcache.log_verbosity_level=1
; 含义：设置 Opcache 日志的详细程度。
;       0：不记录任何 Opcache 错误。
;       1：记录错误（默认值）。
;       2：记录警告。
;       3：记录信息（调试信息）。
;       4：记录调试信息。
; 推荐：生产环境推荐设置为 1 或 2。您这里设置为 1 是合适的。
; 理由：设置为 1 (默认) 足够记录 Opcache 的关键错误，有助于发现配置问题或运行时异常。
;       设置为 2 可以记录警告，提供更多信息，但可能增加日志量。
;       不建议在生产环境设置为 3 或 4，因为会产生大量调试信息，迅速填满日志文件，影响性能。
opcache.log_verbosity_level=1

; opcache.max_wasted_percentage=5
; 含义：当 Opcache 共享内存中“浪费”的内存（例如，由于文件更新导致旧缓存失效，但内存未被立即回收）达到此百分比时，
;       Opcache 会尝试重启自身（如果配置允许），以回收这些浪费的内存。
; 推荐：生产环境推荐开启并保持默认值 5% 或根据实际情况微调。您这里设置为 5 是合适的。
; 理由：防止 Opcache 内存碎片化严重，导致有效缓存空间不足。当浪费的内存达到阈值时，
;       Opcache 会尝试清理，确保缓存效率。5% 是一个合理的平衡点，既能避免频繁重启，又能有效管理内存。
opcache.max_wasted_percentage=5

; opcache.validate_permission=1
; 含义：当 Opcache 检查脚本文件的时间戳时，是否也同时检查文件的权限。
; 推荐：生产环境通常推荐设置为 0 (关闭)。您这里设置为 1 是不推荐的。
; 理由：
;   设置为 1 (开启)：Opcache 会在每次文件时间戳验证时，额外检查文件的权限。这会增加文件系统 I/O 开销，
;                    对性能有轻微负面影响。在生产环境中，文件权限通常是固定的，并且由部署流程保证。
;   设置为 0 (关闭，推荐)：Opcache 不会检查文件权限，只检查时间戳。这可以减少不必要的 I/O 操作，
;                    从而提高性能。前提是您已经通过文件系统权限管理确保了 PHP 脚本文件的正确访问权限。
opcache.validate_permission=0

; 启用并设置二级缓存目录。
; 当共享内存已满、服务器重启或共享内存重置时，它应该能提高性能。
; 默认的 "" 会禁用基于文件的缓存。
; 开启文件缓存作为二级缓存可以在某些情况下提供容错和启动加速。
opcache.file_cache=/tmp/opcache_cache

; 启用或禁用共享内存中的操作码缓存。
; 如果启用了 opcache.file_cache，并且你希望仅从文件缓存加载，可以设置为 1。
;opcache.file_cache_only=0
; 当从文件缓存加载脚本时，启用或禁用校验和验证。
; 开启可以确保文件完整性，但会略微增加开销。在生产环境通常保持开启。
opcache.file_cache_consistency_checks=1

;关闭文档注释缓存（节省内存）
opcache.save_comments=0

; 启用大页内存（需OS支持）
opcache.huge_code_pages=1

; +----------------------------------------------------------------------  
; | JIT   
; +----------------------------------------------------------------------
; 保持tracing模式（最佳选择）
opcache.jit = tracing
; 关键！启用JIT（建议100-200MB）
opcache.jit_buffer_size = 160M
; 提升多态调用处理能力（适应依赖注入）
opcache.jit_max_polymorphic_calls = 8
; 增加递归优化深度（适应路由解析）
opcache.jit_max_recursive_calls = 5
; 增加根跟踪数（复杂框架需要）
opcache.jit_max_root_traces = 1024
; 提升热点函数优化阈值
opcache.jit_hot_func = 192
; 其他性能参数
; 保持开启（提升超全局变量性能）
auto_globals_jit = On
; 保持开启（加速正则匹配）
pcre.jit = On
; 提升内联优化能力
; 增加循环展开优化
opcache.jit_hot_loop = 128
; 提升返回点优化
opcache.jit_hot_return = 16
; 增加跟踪能力
; 适应复杂业务逻辑
opcache.jit_max_side_traces = 128
; 允许更长代码路径优化
opcache.jit_max_trace_length = 1024
```

### opcache.interned_strings_buffer 缓冲区溢出的性能影响
| **指标**                 | **正常状态**  | **溢出状态 (当前)**   | **性能损耗**  |
| ---------------------- | --------- | --------------- | --------- |
| 内存占用                   | 减少 20-30% | 增加 15-25%       | ⬆️ 45-55% |
| 请求处理时间                 | 稳定低延迟     | 波动增大 (+10-30ms) | ⬆️ 15%    |
| CPU 使用率                | 正常水平      | 峰值增加 8-12%      | ⬆️ 10%    |
| GC(Garbage Collection) | 低频触发      | 频繁触发            | ⬆️ 300%   |
## 3.对比测试结果
| 指标           | 开启前    | 开启后    | 变化率    | 优化效果       |
| ------------ | ------ | ------ | ------ | ---------- |
| 平均值（ms）      | 132    | 24     | -82.6% | 响应时间显著降低   |
| 中位数（ms）      | 89     | 18     | -79.8% | 多数请求速度大幅提升 |
| 90% 百分位（ms）  | 291    | 21     | -92.8% | 长尾请求优化显著   |
| 95% 百分位（ms）  | 362    | 27     | -92.5% | 极端情况响应时间减少 |
| 99% 百分位（ms）  | 384    | 190    | -50.5% | 极端情况仍有优化空间 |
| 最大值（ms）      | 386    | 219    | -43.3% | 最慢请求时间缩短   |
| 吞吐量（#/sec）   | 10.17  | 10.12  | -0.5%  | 几乎无变化      |
| 接收数据速率（KB/s） | 344.32 | 342.40 | -0.6%  | 无显著变化      |
| 发送数据速率（KB/s） | 33.85  | 33.67  | -0.5%  | 无显著变化      |
# 九、常见误区与解决方案  
## 1. 误区：OPcache能缓存所有文件  
- **真相**：OPcache仅缓存PHP脚本的字节码，其他文件需通过Redis/Memcached等工具缓存。  
## 2. 误区：OPcache配置越大越好  
- **真相**：内存过大会导致其他服务资源不足，需根据服务器内存合理分配。  
# 十、总结  
OPcache通过**字节码缓存、共享内存复用和代码优化**，显著提升PHP应用的执行效率。合理配置参数（如内存大小、文件数量限制）并结合其他缓存技术（如Redis），可实现更全面的性能优化。在生产环境中，需禁用Xdebug并监控缓存命中率，以确保OPcache发挥最佳效果。  