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

### 3.2 .clang

`Ctrl+Shft+p`配置如下内容,[官网配置](https://clangd.llvm.org/config.html)

![image-20220328225226433](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203282252474.png)

`.clangd`

```yaml
CompileFlags:                             
    Add: 
      [
        -std=c++14,
        -Wno-documentation,
        -Wno-missing-prototypes,
      ]
Diagnostics:
  ClangTidy:
    Add:
    [
        performance-*,
        bugprone-*,
        modernize-*,
        clang-analyzer-*,
        readability-identifier*,
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
    ]
    CheckOptions:
      readability-identifier-naming.VariableCase: camelCase
```

`.clang-format`

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



### 4.2 json配置

```json
{
    "tasks": [
        {
            "type": "shell",
            "label": "g++ build",
            "command": "C:\\Users\\FengisZZZ\\ServerTools\\mingw64\\bin\\g++.exe",
            "args": [
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
        },
        {
            "type": "shell",
            "label": "clang++ build",
            "command": "C:\\Users\\FengisZZZ\\ServerTools\\mingw64\\bin\\clang++.exe",
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

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "gdb",
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
            "preLaunchTask": "g++ build",
        },
        {
            "type": "lldb",
            "request": "launch",
            "name": "lldb",
            "program": "${workspaceFolder}\\build\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "internalConsoleOptions": "neverOpen",
            "preLaunchTask": "g++ build",
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



