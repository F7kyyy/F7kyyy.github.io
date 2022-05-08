---
title: 在vscode上使用clangd
date: 2022-03-28 10:19:38
tags:
    - 环境配置
    - C++
    - Visual Studio Code
categories: 环境配置
math: false
index_img: /img/article/vscode.png
---

首先我们先明白一个概念LSP，额，当然不是老色批🤣

>LSP（Language Server Protocol）开源的语言服务器协定。由RedHat、Microsoft和 Codenvy 联合推出，可以让不同的程序编辑器与集成开发环境（IDE）方便嵌入各种程序语言，允许开发人员在最喜爱的工具中使用各种语言来撰写程序

而C++的LSP有

- ms-vscode.cpptools
- clangd
- ccls

visual studio code 中微软官方的C/C++插件使用的是第一个，经常出现各种各样的问题，同时代码补全不是跟好用，例如C++ STL 在使用时无法使用括号补全。同时经常出现更新不及时，更改代码前出现的报错有时需要重新打开项目才会消失:cry:,就很烦，所以这里使用官方推荐的clangd插件进行C/C++补全，静态检查，高亮功能。

## 1. 备份原有的配置文件

在Windows 10上使用 C/C++调试时默认会自动生成`task.json`和`launch.json`文件，但有的时候会抽风无法生成`launch.json`，这里做一个备份，基本上官方配置复制下来更改一下安装的工具链地址就可以用：[配置地址](https://code.visualstudio.com/docs/cpp/config-mingw)，我做了一些个性化配置。

**task.json 相当于输入一段g++命令，对一段C++代码进行编译和执行，launch.json 会每次调用tasks.json**

使用这个配置可以在调试的时候自动聚焦到终端，生成的二进制文件会出现在根目录的build文件夹

- `task.json`

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: build active file",
            "command": "C:\\Users\\FengisZZZ\\ServerTools\\mingw64\\bin\\g++.exe",
            "args": [
                "-std=c++20",
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${workspaceFolder}\\build\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```

- `launch.json`

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}\\build\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "internalConsoleOptions": "neverOpen",
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\Users\\FengisZZZ\\ServerTools\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: build active file",
        }
    ]
}
```

## 2. 在Ubuntu中设置

### 2.1 安装包

```bash
sudo apt update
sudo apt install g++ gcc gdb cmake make llvm clangd lldb
```

### 2.2 安装vscode 插件

![image-20220328110618664](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203281106759.png)

在安装C/C++和clangd插件后，会出现**冲突问题**，是两个插件补全出现冲突，在`setting.json`中添加，关闭自动补全

```json
// 只第一个其实就可以
"C_Cpp.intelliSenseEngine": "Disabled",
"C_Cpp.autocomplete": "Disabled",
"C_Cpp.errorSquiggles": "Disabled",
```

### 2.3 使用clang++,lldb

配置方法与正常的配置差不多，有些许修改，需要注意在linux下，二进制文件没有“.exe后缀”

- ` tasks.json`

```json
{
    "tasks": [
        {
            "type": "shell",
            "label": "C/C++: build active file",
            "command": "/usr/bin/clang++",
            "args": [
                "-std=c++20",
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${workspaceFolder}/build/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```

- `launch.json`

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            "program": "${workspaceFolder}/build/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "internalConsoleOptions": "neverOpen",
            "preLaunchTask": "C/C++: build active file",
        }
    ]
}
```

### 2.4 使用g++,gdb

因为我们只关闭了C/C++插件的自动补全功能，所以理论上我们是可以使用官方插件进行调试的，使用clangd写代码，C/C++调试。官方的插件在调试上对内存断点上更加好用

![image-20220328165732579](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203281657632.png)

`tasks.json`

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "shell",
            "label": "g++ build active file",
            "command": "/usr/bin/g++",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${workspaceFolder}/build/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "/usr/bin"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

`launch.json`

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "DEBUG",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "internalConsoleOptions": "neverOpen",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "g++ build active file",
            "miDebuggerPath": "/usr/bin/gdb"
        }
    ]
}
```

由于编译和调试所依赖的工具和插件是不同的，我甚至可以使用clang++编译，gdb调试，只需要将`tasks.json`的内容更换为使用clang++时即可

### 2.5 使用Cmake进行多文件编译

打开一个CMake项目，这是一个简单的项目，只有一个头文件和源文件构成

![image-20220328210628497](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203282106596.png)

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(CppClangd)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
add_executable(CppClangd  main.cpp  MyFunction.h)
```

#### 2.5.1 生成compile_command.json

clangd需要根据该文件获取各个文件的include path，以及编译警告错误之类的，必须要有这个文件

- 在CMakeList.txt中添加即可

```cmake
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

- 在build目录下

```cmake
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=YES .
```



#### 2.5.2 Debug配置

实际上，由于我们因为只禁用了C/C++的自动补全功能(再次强调:laughing:),所以正常来讲可以忽略这个东西，但是如果完全不用官方这一套的话，还是需要的。我们只需要调试，所以只需要launch.json,将我们之前的launch.json根据[Cmake Tools](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/debug-launch.md#debug-using-a-launchjson-file)官方文档改一改就可以了

- 使用gdb

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "DEBUG",
            "type": "cppdbg",
            "request": "launch",
            "program": "${command:cmake.launchTargetPath}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "internalConsoleOptions": "neverOpen",
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "miDebuggerPath": "/usr/bin/gdb"
        }
    ]
}
```

- 使用lldb

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            "program": "${command:cmake.launchTargetPath}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "internalConsoleOptions": "neverOpen",
        }
    ]
}
```



## 3. 配置clangd

### 3.1 setting.json

```json
"clangd.arguments": [
        " --compile-commands-dir=${workspaceFold}$/build",
        // clangd在创建索引使用的线程数
        "-j=4",
        // 启用 Clang-Tidy 以提供静态检查
        "--clang-tidy",
        // 建议风格：重载函数只会给出一个建议；反可以设置为detailed
        "--completion-style=bundled",
        // 启用,补全函数时，将会给参数提供占位符
        "--function-arg-placeholders=false",
        // 输入建议中，已包含头文件的项与还未包含头文件的项会以圆点加以区分
        "--header-insertion-decorators",
        // 补充头文件
        "--header-insertion=iwyu",
        // 让 Clangd 生成更详细的日志
        "--log=verbose",
        // pch存储在内存中，内存消耗更大，但是性能更优
        "--pch-storage=memory",
        // 输出的 JSON 文件更美观
        "--pretty",
    	// 设置项目，或者用户config
    	"--enable-config",
    ],

// LLDB 指针显示解引用内容
"lldb.dereferencePointers": true,
// LLDB 鼠标悬停在变量上时预览变量值
"lldb.evaluateForHovers": true,
// LLDB 监视表达式的默认类型
"lldb.launch.expressions": "simple",
// LLDB 不显示汇编代码
"lldb.showDisassembly": "never",
// LLDB 生成更详细的日志
"lldb.verboseLogging": true,

// 保存 cmake.sourceDirectory 或 CMakeLists.txt 内容时，不自动配置 CMake 项目目录
"cmake.configureOnEdit": false,
// 在 CMake 项目目录打开时自动对其进行配置
"cmake.configureOnOpen": true,
// 成功配置后，将 compile_commands.json 复制到此位置
"cmake.copyCompileCommands": "",
```

### 3.2 .clang-tidy

`Ctrl+Shft+p`配置如下内容,[官网配置](https://clangd.llvm.org/config.html)

![image-20220328225226433](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203282252474.png)

可能官网配置较为繁琐，这里列举了大部分的中文含义和示例代码

```c++
Checks:
// 从整数类型转换为字符串默认使用std::to_string
1. boost-use-to-string

// 参数注释
2. bugprone-argument-comment
// before
void foo(int MeaningOfLife);
foo(42);
// after
void foo(int MeaningOfLife);
foo(/*MeaningOfLife=*/42);

// 要求相同分支合并
3. bugprone-branch-clone
// before
switch (ch) {
case 'a':
  return 10;
case 'A':
  return 10;
default:
  return 10;
}
// after
switch (ch) {
case 'a':
case 'A':
  return 10;
default:
  return 10;
}

return test_value(x) ? x : x;

// 基类拷贝构造函数初始化
4. bugprone-copy-constructor-init
class Copyable {
public:
  Copyable() = default;
  Copyable(const Copyable &) = default;
};
class X2 : public Copyable {
  X2(const X2 &other) {} // Copyable(other) is missing
};
class X4 : public Copyable {
  X4(const X4 &other) : Copyable() {} // other is missing
};

// 禁止返回动态初始化的静态变量
5. bugprone-dynamic-static-initializers
int foo() {
  static int k = bar();
  return k;
}

// 禁止在一些函数内抛出异常，避免带来一些风险（析构函数、移动构造函数、移动赋值运算符、main函数、swap函数等）
6. bugprone-exception-escape

// 检查截断和溢出的情况
7. bugprone-fold-init-type
auto a = {0.5f, 0.5f, 0.5f, 0.5f};
return std::accumulate(std::begin(a), std::end(a), 0);
auto a = {65536LL * 65536 * 65536};
return std::accumulate(std::begin(a), std::end(a), 0);

// 检查同名但未定义的命名空间
8. bugprone-forward-declaration-namespace
namespace na { struct A; }
namespace nb { struct A {}; }
nb::A a;

// 排除不正确舍入问题
9. bugprone-incorrect-roundings
(int)(double_expression + 0.5)

// 检查容易出错的无限循环
10. bugprone-infinite-loop

// 检查会导致精度损失的整数除法
11. bugprone-integer-division

// 查找由于缺少括号而可能具有意外行为的宏
12. bugprone-macro-parentheses

// 检查字符串操作的不当行为
13. bugprone-misplaced-operator-in-strlen-in-alloc

// 检查错误的指针动态分配内存
14. bugprone-misplaced-pointer-arithmetic-in-alloc
// before
void bad_malloc(int n) {
  char *p = (char*) malloc(n) + 10;
}
// after
char *p = (char*) malloc(n + 10);

// 类型转换造成的精度损失
15. bugprone-misplaced-widening-cast
// before
return (long)(x * 1000);
// after
return (long)x * 1000;

// 检查std::move()的错误使用
16. bugprone-move-forwarding-reference

// 检查多语句宏定义
17. bugprone-multiple-statement-macro
#define INCREMENT_TWO(x, y) (x)++; (y)++
if (do_increment)
  INCREMENT_TWO(a, b);  // (b)++ will be executed unconditionally.

// 查找容易出错的noescape
18. bugprone-no-escape
void foo(__attribute__((noescape)) int *p) {
  dispatch_async(queue, ^{
    *p = 123;
  });
};

// 检查可能导致非空终止结果的函数调用，使用strcpy()、strncpy()、strcpy_s()、 strncpy_s()代替memcpy()和memcpy_s()
19. bugprone-not-null-terminated-result
static char *stringCpy(const std::string &str) {
  char *result = reinterpret_cast<char *>(malloc(str.size()));
  memcpy(result, str.data(), str.size());
  return result;
}
static char *stringCpy(const std::string &str) {
  char *result = reinterpret_cast<char *>(malloc(str.size() + 1));
  strcpy(result, str.data());
  return result;
}

// 检查对父虚函数的调用
20. bugprone-parent-virtual-call
struct A {
  int virtual foo() {...}
};
struct B: public A {
  int foo() override {...}
};
struct C: public B {
  int foo() override { A::foo(); }
//                     ^^^^^^^^
};

// 检查对pthread_*或posix_*函数调用是否判断负返回值
21. bugprone-posix-return
// before
if (posix_fadvise(...) < 0) {}
// after
if (posix_fadvise(...) > 0) {}

// 检查逻辑if的重复判断冗余
22. bugprone-redundant-branch-condition
if (onFire) {
  if (onFire && peopleInTheBuilding > 0)
    scream();
}

// 检查不规范的下划线使用
23. bugprone-reserved-identifier

// 检查容易出错的信号处理程序
24. bugprone-signal-handler

// 检查错误的字符型与字符型、整型之间的强制转换和比较
25. bugprone-signed-char-misuse

// 检查可能出错的sizeof使用
26. bugprone-sizeof-container
27. bugprone-sizeof-expression
std::string s;
int a = 47 + sizeof(s); // 报错
int b = sizeof(std::string); // 允许

std::string array_of_strings[10];
int c = sizeof(array_of_strings) / sizeof(array_of_strings[0]); // 允许

std::array<int, 3> std_array;
int d = sizeof(std_array); // 允许

class Point {
  [...]
  size_t size() { return sizeof(this); }  // 不允许，可能是sizeof(*this)
  [...]
};

// 检查唤醒函数的错误使用
28. bugprone-spuriously-wake-up-functions

// 检查容易出错的字符串初始化
29. bugprone-string-constructor
std::string str('x', 50); // should be str(50, 'x')

// 检查容易出错的字符串赋值(可能导致误判)
30. bugprone-string-integer-assignment
std::string s;
int x = 5965;
s = 6;	
s = x;

// 检查带有嵌入\0字符的字符串
31. bugprone-string-literal-with-embedded-nul
const char *Bytes[] = "\x03\0x02\0x01\0x00\0xFF\0xFF\0xFF";
std::string str("abc\0def"); 

// 容易出错的枚举用法
32. bugprone-suspicious-enum-usage

// 检查错误的文件包含（只包含头文件）
33. bugprone-suspicious-include.HeaderFileExtensions

// 检查错误的memset的使用
34. bugprone-suspicious-memset-usage
void foo() {
  int i[5] = {1, 2, 3, 4, 5};
  int *ip = i;
  char c = '1';
  char *cp = &c;
  int v = 0;

  // Case 1
  memset(ip, '0', 1); // suspicious
  memset(cp, '0', 1); // OK

  // Case 2
  memset(ip, 0xabcd, 1); // fill value gets truncated
  memset(ip, 0x00, 1);   // OK

  // Case 3
  memset(ip, sizeof(int), v); // zero length, potentially swapped
  memset(ip, 0, 1);           // OK
}

// 检查可疑的逗号丢失
35. bugprone-suspicious-missing-comma
const char* B[] = "This" " is a "    "test"; // 允许
const char* Test[] = {
  "line 1",
  "line 2"     // 报错
  "line 3",
  "line 4",
  "line 5"
};

// 检查容易出错的可以分号
36. bugprone-suspicious-semicolon
if (x < y);
{
  x++;
}

// 检查可疑的字符串比较
37. bugprone-suspicious-string-compare
if (strcmp(...))       // 建议显式比较
if (!strcmp(...))      // 允许
if (strcmp(...) != 0)  // 允许
if (strcmp(...) == -1) // 错误
if (strcmp(...) < 0.)  // 错误

// 检查容易出错的交换参数
38. bugprone-swapped-arguments

// 检查do-while常为false时的continue使用
39. bugprone-terminating-continue
do {
  ...
  continue; // 直接终止循环
  ...
} while(false);

// 检查可能缺失的throw关键字
40. bugprone-throw-keyword-missing
void f(int i) {
  if (i < 0) {
    // Exception is created but is not thrown.
    std::runtime_error("Unexpected argument");
  }
}

// 检查容易出错的循环变量
41. bugprone-too-small-loop-variable
int main() {
  long size = 294967296l;
  for (short i = 0; i < size; ++i) {}
}
int doSomething(const std::vector& items) {
  for (short i = 0; i < items.size(); ++i) {}
}

// 查找容易出错的未定义内存操作
42. bugprone-undefined-memory-manipulation

// 检查容易出错的委托构造函数
43. bugprone-undelegated-constructor

// 检查未对new进行异常处理
44. bugprone-unhandled-exception-at-new
int *f() noexcept {
  int *p = new int[1000];
  // ...
  // std::bad_alloc
  return p;
}

// 检查容易出错的未处理自赋值
55. bugprone-unhandled-self-assignment

// 检查看起来像RAII对象的临时对象。
56. bugprone-unused-raii

// 检查未使用的一些返回值（如返回指针）
57. bugprone-unused-return-value

// 禁止使用std::move之后的对象仍被使用
58. bugprone-use-after-move

// 检查函数名与基类虚函数类似的函数声明
59. bugprone-virtual-near-miss

// 会报未知的警告
60. cert-dcl21-cpp
61. cert-dcl50-cpp
62. cert-env33-c

// 检查对std posix等命名空间的修改
61. cert-dcl58-cpp

// 检查不验证转换字符串到数字的有效性的代码，如c语言atoi()
62. cert-err34-c

// 检查浮点类型的循环
63. cert-flp30-c

// 生成动态类型信息
64. clang-analyzer-core.DynamicTypePropagation

// 检查未初始化的块
65. clang-analyzer-core.uninitialized.CapturedBlockVariable

// 检查释放后的内部指针的使用
66. clang-analyzer-cplusplus.InnerPointer

// 检查从非空返回类型 返回 可能空的指针
67. clang-analyzer-nullability.NullableReturnedFromNonnull

68. clang-analyzer-optin.osx.OSObjectCStyleCast
69. clang-analyzer-optin.performance.GCDAntipattern
70. clang-analyzer-osx.MIG

// 检查过度填充的结构
71. clang-analyzer-optin.performance.Padding

72. clang-analyzer-osx.OSObjectRetainCount

// 检查objc
73. clang-analyzer-osx.ObjCProperty 
74. clang-analyzer-osx.cocoa.AutoreleaseWrite

// cocoa相关
75. -clang-analyzer-osx.cocoa-*

// va_lists相关
76. clang-analyzer-valist-

// mt安全
77. concurrency-mt-unsafe

// 并发线程
78. concurrency-thread-canceltype-asynchronous

// 避免使用goto
79. cppcoreguidelines-avoid-goto

// 禁止非常量全局变量
80. cppcoreguidelines-avoid-non-const-global-variables

// 检查extern对象的全局变量的初始值设定项初始化顺序问题
81. cppcoreguidelines-interfaces-global-init

// 检查可能被认为有问题的宏用法
82. -cppcoreguidelines-macro-usage

83. -cppcoreguidelines-narrowing-conversions

// 对c风格内存分配的检查
84. cppcoreguidelines-no-malloc

// 检查内存分配时是否使用gsl的方法
85. -cppcoreguidelines-owning-memory

86. cppcoreguidelines-prefer-member-initializer
87. cppcoreguidelines-pro-bounds-array-to-pointer-decay
88. cppcoreguidelines-pro-bounds-constant-array-index

// 标记指针的算法使用
89. -cppcoreguidelines-pro-bounds-pointer-arithmetic

// 标记const_cast的使用
90. -cppcoreguidelines-pro-type-const-cast
91. -cppcoreguidelines-pro-type-cstyle-cast

92. -cppcoreguidelines-pro-type-member-init

// 标记reinterpret_cast的使用
93. -cppcoreguidelines-pro-type-reinterpret-cast

// 标记static_cast的使用
94. -cppcoreguidelines-pro-type-static-cast-downcast 

// 标记union成员的访问权限
95. -cppcoreguidelines-pro-type-union-access

// 标记对c样式可变参数函数调用以及对va_arg的使用
96. cppcoreguidelines-pro-type-vararg

// 检查导致slice的情况
97. cppcoreguidelines-slicing
struct B { int a; virtual int f(); };
struct D : B { int b; int f() override; };
void use(B b) {  // Missing reference, intended?
  b.f();  // Calls B::f.
}

D d;
use(d);  // Slice

98. cppcoreguidelines-special-member-functions
99. darwin-avoid-spinlock
100. darwin-dispatch-once-nonstatic

// 使用默认参数时发出警告
101. -fuchsia-default-arguments-calls

// 函数定义默认参数时发出警告
102. -fuchsia-default-arguments-declarations

// 继承自多个非纯虚类发出警告
103. -fuchsia-multiple-inheritance

// 检查已被重载的运算符
104. -fuchsia-overloaded-operator

// 限制创建静态对象（除非构造函数是constexpr类型或没有显式构造函数）
105. fuchsia-statically-constructed-objects

106. fuchsia-trailing-return

// 对虚继承类发出警告(报错)
107. fuchsia-virtual-inheritance
class B : public virtual A {};

// 检查make_pair是否推断出模板类型
108. google-build-explicit-make-pair

// 在标头中查找匿名命名空间
109.google-build-namespaces
- key:google-build-namespaces.HeaderFileExtensions
    value:h,hh,hpp,hxx

// 检查是否使用using namespace
110. google-build-using-namespace

// 为虚函数参数添加默认值
111. google-default-arguments

// 检查显式构造函数，避免隐式转换带来的风险
112. -google-explicit-constructor

// 在头文件中标记全局命名空间污染
113. google-global-names-in-headers
  - key: google-global-names-in-headers.HeaderFileExtensions
    value: h

// objc
114. -google-objc-avoid-nsobject-new
115. -google-objc-avoid-throwing-exception
116. -google-objc-function-naming
117. -google-objc-global-variable-declaration

// 检查googletest的测试以及测试示例名称是否有下划线（测试名称和测试用例名称中不允许使用下划线）
118. google-readability-avoid-underscore-in-googletest-name

// 查找 C 风格强制转换的用法
119. google-readability-casting

// 查找TODO有没有署用户名、邮件等
120. google-readability-todo
// TODO(kl@gmail.com): Use a "*" here for concatenation operator.
// TODO(Zeke) change this to use relations.
// TODO(bug 12345): remove the "Last visitors" feature.

// 检查整数定义，将short, long，long long改为intxx_类型
121. -google-runtime-int

// 检查用户自定义表述的重载运算符
121. google-runtime-operator

// 用户自定义函数名中，将带有case字符改为suite
122. google-upgrade-googletest-case

// 要求goto只跳过块的一部分（不使用goto）
123. -hicpp-avoid-goto

// 确保throw表达式中的每个值都是std::exception的实例
124. hicpp-exception-baseclass

// 有if和else if一定要包含else（即使内容为空），有switch要包含default，少量case用if代替。
125. hicpp-multiway-paths-covered

// 检查汇编语句
126. hicpp-no-assembler

// 检查对有符号整数类型的按位运算的使用，避免风险
127. hicpp-signed-bitwise

// 检查linux内核代码
128. linuxkernel-must-use-errs

// 检查不符合LLVM风格的头文件
129. llvm-header-guard
  - key: llvm-header-guard.HeaderFileExtensions
    value: ',h,hh,hpp,hxx'

// 检查include的正确顺序（llvm风格）
130. llvm-include-order

// 长命名空间结束注释
131. llvm-namespace-comment

// cast的一些变换
132. llvm-prefer-isa-or-dyn-cast-in-conditionals
// before
if (auto x = cast<X>(y)) {}
// after
if (auto x = dyn_cast<X>(y)) {}
// befo
if (cast<X>(y)) {}
// after
if (isa<X>(y)) {}

// unsigned用Register代替
133. llvm-prefer-register-over-unsigned

// Twine方法使用后面加.str()
134. llvm-twine-local
// before
static Twine Moo = Twine("bark") + "bah";
// after
static std::string Moo = (Twine("bark") + "bah").str();

// 检查调用解析为__llvm_libc命名空间内的函数
135. llvmlibc-callee-namespace

136. llvmlibc-implementation-in-namespace

// 查找编译器未提供的系统 libc 头文件,如stdio.h
137. llvmlibc-restrict-system-libc-headers

// 在头文件中查找非 extern 非内联函数和变量定义
138. -misc-definitions-in-headers
int a = 1; //全局，不允许
inline int e() { //允许
  return 1;
}

// 检查const的错误使用
139. misc-misplaced-const
typedef int *int_ptr;
void f(const int_ptr ptr) {
  *ptr = 0; // 意外的修改了数值
  ptr = 0; // 编译不通过
}

// 检查new和delete
140. misc-new-delete-overloads

// 检查函数递归错误
141. misc-no-recursion

// 检查一些对象，不允许复制，比如FILE
142. misc-non-copyable-objects

// 规定类成员变量必须为private
143. -misc-non-private-member-variables-in-classes 

// 检查冗余表达式
144. misc-redundant-expression
(p->x == p->x);
(p->x < p->x);

// 用static_assert()代替assert()
145. misc-static-assert

// 查找违反“按值抛出，按引用捕捉”的规则
146. misc-throw-by-value-catch-by-reference

// 检查重载运算符的错误返回类型
147. misc-unconventional-assign-operator

// 将unique_ptr::reset(release())替换为std::move()
148. misc-uniqueptr-reset-release

// 查找未使用的命名空间别名声明
149. misc-unused-alias-decls

// 查找未使用的参数并报错（可能代码有误）
150. misc-unused-parameters

// 查找未使用的using声明
151. misc-unused-using-decls

// 替换std::bind()的使用
152. modernize-avoid-bind

// 避免使用c语言的array
153. modernize-avoid-c-arrays
int a[] = {1, 2};//报错

// 嵌套命名空间的简化（可以简化时）
154. modernize-concat-nested-namespaces
// before
namespace n1 {
namespace n2 {
void t();
}
}
// after
namespace n1::n2 {
void t();
}

// 检查c++中弃用的c头文件
155. modernize-deprecated-headers

// 检查弃用的类方法
156. modernize-deprecated-ios-base-aliases
弃用								替代
std::ios_base::io_state			std::ios_base::iostate
std::ios_base::open_mode		std::ios_base::openmode
std::ios_base::seek_dir			std::ios_base::seekdir
std::ios_base::streamoff	 
std::ios_base::streampos	 

// 避免有风险的循环，替换为c++11风格
157. modernize-loop-convert

// 对于share_ptr的一些更改
158. modernize-make-shared
// before
auto my_ptr = std::shared_ptr<MyPair>(new MyPair(1, 2));
my_ptr.reset(new MyPair(1, 2));
// after
auto my_ptr = std::make_shared<MyPair>(1, 2);
my_ptr = std::make_shared<MyPair>(1, 2);

// 对于unique_ptr的一些更改
159. modernize-make-unique
// before
auto my_ptr = std::unique_ptr<MyPair>(new MyPair(1, 2));
my_ptr.reset(new MyPair(1, 2));
// after
auto my_ptr = std::make_unique<MyPair>(1, 2);
my_ptr = std::make_unique<MyPair>(1, 2);

160. modernize-pass-by-value

// 字符串初始化的风格检查
161. modernize-raw-string-literal

// 检查冗余的void
162. modernize-redundant-void-arg
int fun(void)

// 用unique_ptr替换auto_ptr
163. modernize-replace-auto-ptr

164. modernize-replace-disallow-copy-and-assign-macro

// 将std::random_shuffle替换为std::shuffle
165. modernize-replace-random-shuffle

// 用花括号初始化器列表替换返回中对构造函数的显式调用
166. modernize-return-braced-init-list
// before
Foo bar() {
  Baz baz;
  return Foo(baz);
}
// after
Foo bar() {
  Baz baz;
  return {baz};
}

// 关于缩放的使用
167. modernize-shrink-to-fit

// 关于static_assert空字符串的检查
168. modernize-unary-static-assert

// 检查auto的使用风格
169. modernize-use-auto

// 检查bool类型，用true和false表示
170. modernize-use-bool-literals

// 构造函数初始化的设定值风格
171. modernize-use-default-member-init
// before
A() : i(5), j(10.0)
int i;
double j;
// after
A() {}
int i{5};
double j{10.0};

// 在std::vector、std::deque、std::list中插入的使用限制
172. -modernize-use-emplace

// 将特殊成员函数的默认主体替换为显式默认函数声明
173. modernize-use-equals-default
// before
A() {}
// after
A() = default;

// 将删除的特殊成员函数显式表示
174. modernize-use-equals-delete
A(const A&) = delete;	

// const成员函数的使用风格
175. -modernize-use-nodiscard

// 函数定义的throw替换为noexcept
176. modernize-use-noexcept
// before
void foo() throw();
// after
void foo() noexcept;

// 将空指针NULL改为nullptr
177. -modernize-use-nullptr

// 在重载虚函数中添加override声明
178. modernize-use-override

// 使用尾随返回类型
179. -modernize-use-trailing-return-type

180. modernize-use-transparent-functors
181. modernize-use-uncaught-exceptions

// 将typedef转换为using
182. modernize-use-using
// before
typedef int variable;
// after
using variable = int;

// 关于消息传递接口的检查
183. mpi-buffer-deref

// 检查消息传递接口不匹配问题
184. mpi-type-mismatch

// objc相关
185. objc-avoid-nserror-init	 
186. objc-dealloc-in-category	 
187. objc-forbidden-subclassing	 
188. objc-missing-hash	 
189. objc-nsinvocation-argument-lifetime
190. objc-property-declaration
191. objc-super-self

// openmp相关
192. openmp-exception-escape	 
193. openmp-use-default-none

// 优化find的效率
194. performance-faster-string-find
// before
str.find("A");
// after
str.find('A');

// 不使用for循环里使用auto的警告
195. performance-for-range-copy
  - key:             performance-for-range-copy.WarnOnAllAutoCopies
    value:           '0'

// for循环中，变量的隐式表示（auto）代替显式表示
196. performance-implicit-conversion-in-loop

// 检查关联容器中的低效使用
197. performance-inefficient-algorithm
// before
auto it = std::find(s.begin(), s.end(), 43);
// after
auto it = s.find(43);

// 检查性能低效的字符串连接，字符串拼接改为append，而不是操作符+
198. performance-inefficient-string-concatenation

// 查找可能导致不必要的内存重新分配的低效vector操作
199. -performance-inefficient-vector-operation

// 关于std::move的一些警告
200. performance-move-const-arg

// 标记移动构造函数初始化
201. performance-move-constructor-init

202. performance-no-automatic-move

// 限制整数到指针的转换
203. performance-no-int-to-ptr

// 检查移动构造函数没有标记noexcept或者为false的情况
204. performance-noexcept-move-constructor

// 检查外部可破坏析构函数的情况
205. performance-trivially-destructible

// 数学库，从c到c++的转换
206. performance-type-promotion-in-math-fn

207. performance-unnecessary-copy-initialization

// 使用参数的移动(std::move)，而不是复制
208. performance-unnecessary-value-param

// 检查以选择性地允许或禁止系统标头的可配置列表
209. portability-restrict-system-includes

210. portability-simd-intrinsics

// 检查函数声明是否具有顶级参数 const
211. -readability-avoid-const-params-in-decls

// 在逻辑语句是否添加大括号
212. readability-braces-around-statements
// before
if (condition)
  statement;
// after
if (condition) {
  statement;
}

// 常量返回值的限制
213. readability-const-return-type

// 检查对size()调用是否可以用empty()来代替
214. readability-container-size-empty

// 将不使用this的非静态成员函数转换为静态成员函数
215. readability-convert-member-functions-to-static

// 查找检查指针是否存在的语句
216. -readability-delete-null-pointer

// 检查多余的else使用
217. readability-else-after-return
// before
if (Value == 1) {
  return;
} else {
  Local++;
}
// after
if (Value == 1) {
  return;
}
Local++;

// 检查功能认知复杂性指标
218. readability-function-cognitive-complexity

// 根据各种指标检查大型函数
219. readability-function-size
// 限制函数的控制语句数
- key:             readability-function-size.BranchThreshold
value:           '-1'
// 限制函数的行数
- key:             readability-function-size.LineThreshold
value:           '-1'
// 限制函数的参数数量
- key:             readability-function-size.ParameterThreshold
value:           '-1'
// 限制函数的语句数
- key:             readability-function-size.StatementThreshold
value:           '800'

// 标识符命名方式限制
220. readability-identifier-naming

// 检查bool与int之间的转换
221. readability-implicit-bool-conversion

// 查找参数名称不一致的声明
222. readability-inconsistent-declaration-parameter-name
// in foo.hpp:
void foo(int a, int b, int c);
// in foo.cpp:
void foo(int d, int e, int f); 

// 将连续的变量声明拆分成逐一声明
223. readability-isolate-declaration
// before
int * pointer = nullptr, value = 42, * const const_ptr = &value;
// after
int * pointer = nullptr;
int value = 42;
int * const const_ptr = &value;

// magic numbers检查
224. readability-magic-numbers

// 检查可以被定义为const的非静态成员函数
225. readability-make-member-function-const

// 检查误导性缩进
226. readability-misleading-indentation
// Dangling else:
if (cond1)
  if (cond2)
    foo1();
else
  foo2();

// 检查出错的索引
227. readability-misplaced-array-index
void f(int *X, int Y) {
  Y[X] = 0;
}

// 查找带有未命名参数的函数
228. readability-named-parameter

// 检查可以更改为指向常量类型的指针类型的函数参数
229. readability-non-const-parameter

// 将指针限定添加到auto推导为指针的类型变量
230. readability-qualified-auto
// before
for (auto Data : MutatablePtrContainer) {
  change(*Data);
}
for (auto Data : ConstantPtrContainer) {
  observe(*Data);
}
// after
for (auto *Data : MutatablePtrContainer) {
  change(*Data);
}
for (const auto *Data : ConstantPtrContainer) {
  observe(*Data);
}

// 删除冗余的访问类型说明
231. readability-redundant-access-specifiers

// 删除无用的return和continue
232. readability-redundant-control-flow

// 删除冗余的变量和函数声明
233. readability-redundant-declaration

// 删除多余的指针冗余
234. readability-redundant-function-ptr-dereference
// before
int f(int,int);
int (*p)(int, int) = &f;
int i = (**p)(10, 50);
// after
int f(int,int);
int (*p)(int, int) = &f;
int i = (*p)(10, 50);

// 查找多余的成员初始化
235. readability-redundant-member-init

// 查找潜在的冗余预处理器指令
236. readability-redundant-preprocessor

// 删除对智能指针.get()方法的冗余调用
237. readability-redundant-smartptr-get

// 检查不必要的调用：std::string::c_str()和std::string::data()
238. readability-redundant-string-cstr

// 查找不必要的字符串初始化
239. readability-redundant-string-init

// 简化对bool类型的使用
240. readability-simplify-boolean-expr
// before
if (b == true)	
// after
if (b)

// 简化下标表达式
241. readability-simplify-subscript-expr
// before
char c = s.data()[i];  
// after
char c = s[i];

// 优化类静态成员的访问方式
242. readability-static-accessed-through-instance
// before
C *c1 = new C();
c1->foo();
c1->x;
// after
C *c1 = new C();
C::foo();
C::x;

// 检查匿名命名空间中的静态成员定义
243. readability-static-definition-in-anonymous-namespace

// 优化字符串比较的使用，尽量使用==或者!=
244. readability-string-compare
if (str1.compare(str2))
if (!str1.compare(str2))

// 传参缩写检查
245. readability-suspicious-call-argument

// 替换<unique_ptr>.release()为<unique_ptr> = nullptr
246. readability-uniqueptr-delete-release
// before
std::unique_ptr<int> P;
delete P.release();
// after
std::unique_ptr<int> P;
P = nullptr;

// 要求整数后缀为大写字符
247. readability-uppercase-literal-suffix
// before
auto x = 1u;
// after
auto x = 1U;

248. readability-use-anyofallof
249. zircon-temporary-objects
```



`.clangd`

```yaml
CompileFlags:                             
    Add: 
      [
        -std=c++20,
        -Wno-documentation,
        -Wno-missing-prototypes
      ]
Diagnostics:
  ClangTidy:
    Add:
    [
        performance-*,
        bugprone-*,
        modernize-*,
        clang-analyzer-*,
        modernize-use-emplace,
    ]
    Remove:
    [
        abseil*,
        fuchsia*,
        llvmlib*,
        zircon*,
        altera*,
        google-readability-todo,
        readability-braces-around-statements,
        hicpp-braces-around-statements,
        modernize-use-trailing-return-type,
        bugprone-easily-swappable-parameters,
        modernize-use-default-member-init
    ]

```

### 3.3 .clang-format

```
# 语言: None, Cpp, Java, JavaScript, ObjC, Proto, TableGen, TextProto
Language: Cpp
# BasedOnStyle: LLVM

# 访问说明符(public、private等)的偏移
AccessModifierOffset: -4

# 开括号(开圆括号、开尖括号、开方括号)后的对齐: Align, DontAlign, AlwaysBreak(总是在开括号后换行)
AlignAfterOpenBracket: Align

# 连续赋值时，对齐所有等号
AlignConsecutiveAssignments: false

# 连续声明时，对齐所有声明的变量名
AlignConsecutiveDeclarations: false

# 右对齐逃脱换行(使用反斜杠换行)的反斜杠
AlignEscapedNewlines: Right

# 水平对齐二元和三元表达式的操作数
AlignOperands: true

# 对齐连续的尾随的注释
AlignTrailingComments: true

# 不允许函数声明的所有参数在放在下一行
AllowAllParametersOfDeclarationOnNextLine: false

# 不允许短的块放在同一行
AllowShortBlocksOnASingleLine: true

# 允许短的case标签放在同一行
AllowShortCaseLabelsOnASingleLine: true

# 允许短的函数放在同一行: None, InlineOnly(定义在类中), Empty(空函数), Inline(定义在类中，空函数), All
AllowShortFunctionsOnASingleLine: None

# 允许短的if语句保持在同一行
AllowShortIfStatementsOnASingleLine: true

# 允许短的循环保持在同一行
AllowShortLoopsOnASingleLine: true

# 总是在返回类型后换行: None, All, TopLevel(顶级函数，不包括在类中的函数), 
# AllDefinitions(所有的定义，不包括声明), TopLevelDefinitions(所有的顶级函数的定义)
AlwaysBreakAfterReturnType: None

# 总是在多行string字面量前换行
AlwaysBreakBeforeMultilineStrings: false

# 总是在template声明后换行
AlwaysBreakTemplateDeclarations: true

# false表示函数实参要么都在同一行，要么都各自一行
BinPackArguments: true

# false表示所有形参要么都在同一行，要么都各自一行
BinPackParameters: true

# 大括号换行，只有当BreakBeforeBraces设置为Custom时才有效
BraceWrapping:
  # class定义后面
  AfterClass: false
  # 控制语句后面
  AfterControlStatement: false
  # enum定义后面
  AfterEnum: false
  # 函数定义后面
  AfterFunction: false
  # 命名空间定义后面
  AfterNamespace: false
  # struct定义后面
  AfterStruct: false
  # union定义后面
  AfterUnion: false
  # extern之后
  AfterExternBlock: false
  # catch之前
  BeforeCatch: false
  # else之前
  BeforeElse: false
  # 缩进大括号
  IndentBraces: false
  # 分离空函数
  SplitEmptyFunction: false
  # 分离空语句
  SplitEmptyRecord: false
  # 分离空命名空间
  SplitEmptyNamespace: false

# 在二元运算符前换行: None(在操作符后换行), NonAssignment(在非赋值的操作符前换行), All(在操作符前换行)
BreakBeforeBinaryOperators: NonAssignment

# 在大括号前换行: Attach(始终将大括号附加到周围的上下文), Linux(除函数、命名空间和类定义，与Attach类似), 
#   Mozilla(除枚举、函数、记录定义，与Attach类似), Stroustrup(除函数定义、catch、else，与Attach类似), 
#   Allman(总是在大括号前换行), GNU(总是在大括号前换行，并对于控制语句的大括号增加额外的缩进), WebKit(在函数前换行), Custom
#   注：这里认为语句块也属于函数
BreakBeforeBraces: Custom

# 在三元运算符前换行
BreakBeforeTernaryOperators: false

# 在构造函数的初始化列表的冒号后换行
BreakConstructorInitializers: AfterColon

#BreakInheritanceList: AfterColon

BreakStringLiterals: false

# 每行字符的限制，0表示没有限制
ColumnLimit: 0

CompactNamespaces: true

# 构造函数的初始化列表要么都在同一行，要么都各自一行
ConstructorInitializerAllOnOneLineOrOnePerLine: false

# 构造函数的初始化列表的缩进宽度
ConstructorInitializerIndentWidth: 4

# 延续的行的缩进宽度
ContinuationIndentWidth: 4

# 去除C++11的列表初始化的大括号{后和}前的空格
Cpp11BracedListStyle: true

# 继承最常用的指针和引用的对齐方式
DerivePointerAlignment: false

# 固定命名空间注释
FixNamespaceComments: true

# 缩进case标签
IndentCaseLabels: false

IndentPPDirectives: None

# 缩进宽度
IndentWidth: 4

# 函数返回类型换行时，缩进函数声明或函数定义的函数名
IndentWrappedFunctionNames: false

# 保留在块开始处的空行
KeepEmptyLinesAtTheStartOfBlocks: false

# 连续空行的最大数量
MaxEmptyLinesToKeep: 1

# 命名空间的缩进: None, Inner(缩进嵌套的命名空间中的内容), All
NamespaceIndentation: None

# 指针和引用的对齐: Left, Right, Middle
PointerAlignment: Right

# 允许重新排版注释
ReflowComments: true

# 允许排序#include
SortIncludes: false

# 允许排序 using 声明
SortUsingDeclarations: false

# 在C风格类型转换后添加空格
SpaceAfterCStyleCast: false

# 在Template 关键字后面添加空格
SpaceAfterTemplateKeyword: true

# 在赋值运算符之前添加空格
SpaceBeforeAssignmentOperators: true

# SpaceBeforeCpp11BracedList: true

# SpaceBeforeCtorInitializerColon: true

# SpaceBeforeInheritanceColon: true

# 开圆括号之前添加一个空格: Never, ControlStatements, Always
SpaceBeforeParens: ControlStatements

# SpaceBeforeRangeBasedForLoopColon: true

# 在空的圆括号中添加空格
SpaceInEmptyParentheses: false

# 在尾随的评论前添加的空格数(只适用于//)
SpacesBeforeTrailingComments: 1

# 在尖括号的<后和>前添加空格
SpacesInAngles: false

# 在C风格类型转换的括号中添加空格
SpacesInCStyleCastParentheses: false

# 在容器(ObjC和JavaScript的数组和字典等)字面量中添加空格
SpacesInContainerLiterals: true

# 在圆括号的(后和)前添加空格
SpacesInParentheses: false

# 在方括号的[后和]前添加空格，lamda表达式和未指明大小的数组的声明不受影响
SpacesInSquareBrackets: false

# 标准: Cpp03, Cpp11, Auto
Standard: Auto

# tab宽度
TabWidth: 4

# 使用tab字符: Never, ForIndentation, ForContinuationAndIndentation, Always
UseTab: Never
```



## 4. Windows上配置

### 4.1 工具链安装

不知道大家有没有注意过自己用的工具链版本

较为知名的几个项目:

- MinGW-w64项目多年不更新，gcc版本停留在8.1,与linux下ubuntu20.04 LTS 9.0 ,Arch 11.2差的很远；
- tdm-gcc在停更了好多年之后，更新了10.0版本

推荐以下两种方式

- [Winlib](https://winlibs.com/) 主要提供GCC 和 LLVM 在windows下的 使用 基本上上游更新，作者就会编译发布

  <img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203301905350.png" alt="image-20220330190524242" style="zoom:67%;" />

  因为是LLVM官方在windows上似乎只提供了MSVC版本的，所以可能存在问题，我目前使用的版本lldb就存在python依赖问题

- [MYSY2](https://www.msys2.org/) 模拟Linux环境，使用Pacman 作为包管理器，作为一个曾经在物理机上安装Arch Linux 的人，Pacman 的强大让我念念不忘，可惜的是Linux下没有好用的QQ,微信。Pacman 中有最新的软件包和几乎永远不需要担心依赖和编译问题，在配置环境中带来巨大痛苦的OpenGL,OpenCV,Eigen，甚至是hadoop的安装只需要一条命令，已经有人为你做好了一切，包管理器让更新变得十分容易。

  <img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202205081312305.png" alt="image-20220508131232171" style="zoom:67%;" />

使用msys2安装工具链的时候，`因为clang 和 mingw-w64是两套完全不同的工具链，分别在mingw64，clang64目录下，而且第三方包并不共享，因此推荐安装mingw-w64版的clang,llvm`

例如使用tdmgcc64（一个mingw-ww64的windows发行版） 编译的opencv在使用clang64文件夹下的clang++是无法通过编译的

```bash
# mingw-w64工具链
pacman -S --needed base-devel mingw-w64-x86_64-toolchain
# 包含clang,clang++等工具
mingw-w64-x86_64-clang
# clangd clang-tidy等工具
mingw-w64-x86_64-clang-tools-extra
# llvm相关
mingw-w64-x86_64-llvm
```

安装后发现clang,clang++,clangd都在`./mingw64/bin`目录下,`./clang64`下都是空目录

记得添加环境变量

<img src="C:\Users\FengisZZZ\AppData\Roaming\Typora\typora-user-images\image-20220508131557192.png" alt="image-20220508131557192" style="zoom:67%;" />

### 4.2 json配置

因为上述安装的包都是为了mingw-w64工具链而服务，所以我们可以使用clang++编译，gdb调试；正常安装clang,mingw-w64则不能如此。

同时还有意外惊喜:smile:，就是可以将cpp文件命名为中文，只用g++,gdb不能正常打断点调试，而用clang++是可以的。

以下是`使用c/c++ 插件`的配置，分别表示使用g++编译，和clang++编译。

```json
{
    "tasks": [
        {
            "type": "shell",
            "label": "g++ build",
            "command": "C:\\Users\\FengisZZZ\\ServerTools\\msys\\mingw64\\bin\\g++.exe",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-Wall",
                "-o",
                "${workspaceFolder}\\bin\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "echo": false,
                "panel": "shared",
                "reveal": "always",
                "focus": true
            },
        },
        {
            "type": "shell",
            "label": "clang++ build",
            "command": "C:\\Users\\FengisZZZ\\ServerTools\\msys\\mingw64\\bin\\clang++.exe",
            "args": [
                "-g",
                "${file}",
                "-Wall",
                "-o",
                "${workspaceFolder}\\bin\\leetcode.exe"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "echo": false,
                "panel": "shared",
                "reveal": "always",
                "focus": true
            },
        }
    ],
    "version": "2.0.0"
}
```

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "gcc debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}\\bin\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "internalConsoleOptions": "neverOpen",
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\Users\\FengisZZZ\\ServerTools\\msys\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "g++ build",
        },
        {
            "name": "clang debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}\\bin\\leetcode.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "internalConsoleOptions": "neverOpen",
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\Users\\FengisZZZ\\ServerTools\\msys\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "clang++ build",
        }
    ]
}
```



`未完 待续`

## 附

`参考链接:`

- Bilibili

[ VS Code + Clangd + CMake 搭建 C/C++开发环境](https://www.bilibili.com/video/BV1sW411v7VZ?p=1)

- CSDN:

[VSCode 配置 C/C++：VSCode + Clang + Clangd + LLDB + CMake + Git](https://blog.csdn.net/tyKuGengty/article/details/120119820)

[vscode + clangd 开发 c/c++](https://blog.csdn.net/weixin_43862847/article/details/119274382)



