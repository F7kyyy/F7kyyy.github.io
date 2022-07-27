---
title: vscode 配置msvc编译器 使用vcpkg进行C++包管理
date: 2022-06-27 21:58:53
tags:
    - 环境配置
    - C++
    - Visual Studio Code
    - Visual Studio
categories: 环境配置
math: false
index_img: https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627221709538.png
---
>主要是还在考试的过程中，完全不想准备考研，在知乎刷到了一篇关于现代c++的介绍，介绍了一些流行的第三方库，如google 的 `abseil`,facebook 的`folly`,所以想安装试一试。一般都需要编译安装，非常的麻烦，所以考虑使用msys2来安装，但是`folly`库msys2居然都没有,果然Windows 还是老老实实的用MSVC吧✌。Vcpkg经过几年的发展已经完全可用了，开源库都有Vcpkg的安装方式。



### 1. 安装Visual Studio 2022

我们可以选择只安装[MS Build Toos](https://visualstudio.microsoft.com/zh-hans/downloads/),但是当我们选择最基本的C++开发组件，大概需要4G左右存储空间，语言包需要额外选上英文，这是Vcpkg依赖。

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627223858130.png" alt="image-20220627223858130" style="zoom:50%;" />

不过，我们为什么不选择直接安装Visual Studio 2022 ，体验一下宇宙第一IDE，毕竟完整安装也只要8G左右空间。**强烈建议直接安装完整版，因为只安装开发工具的话想要再次完整安装会重新下载另外一份开发工具，十分扯淡**，同样的`语言包需要额外选上英文`

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627224405597.png" alt="image-20220627224405597" style="zoom:50%;" />

### 2. 安装Vcpkg

[Github 官网有完整教程](https://github.com/microsoft/vcpkg/blob/master/README_zh_CN.md#%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B-windows)

#### 2.1 安装

官方文档建议安装在`C:\src`目录

```bash
git clone https://github.com/microsoft/vcpkg
.\vcpkg\bootstrap-vcpkg.bat
```

建议将`C:\src\vcpkg`加入环境变量，这样就可以在任意位置使用vcpkg命令

#### 2.2 基本操作

```powershell
# 搜索
vcpkg search [search term]
# 安装
vcpkg install [packages to install]
```

例如安装格式化库`fmt`，因为C++编译器的变化，直接下载二进制包是无法使用的，所以vcpkg相当于是将依赖源码全都下载然后编译，所以比较大的库时间还是比较长，比如OpenCV就需要可能20分钟，小的库也就几十秒

```powershell
vcpkg search fmt
# 指定系统版本
vcpkg install fmt:x64-windows
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627225537803.png" alt="image-20220627225537803" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627225902749.png" alt="image-20220627225902749" style="zoom:50%;" />

### 3.集成在VS中

#### 3.1 全局集成

执行下命令，直接新建一个VS项目，即可使用。

```powershell
vcpkg integrate install
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627231533379.png" alt="image-20220627231533379" style="zoom: 40%;" />

#### 3.2 工程集成和CMake集成

详见github

### 4. 在VS code Clion中使用

这个项目是同时使用`OpenCV`,`fmt`库

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627233132298.png" alt="image-20220627233132298" style="zoom: 40%;" />

`测试代码`

```shell
# opencv太大可以只测试前两个
vcpkg install fmt:x64-windows abseil:x64-windows opencv:x64-windows
```

```c++
#include <iostream>
#include <vector>
//#include <opencv2/core/core.hpp>
//#include <opencv2/highgui/highgui.hpp>
//#include <opencv2/core/utils/logger.hpp>
#include <fmt/format.h>
#include <absl/strings/str_format.h>
#include <absl/strings/str_join.h>

void formatxx() {
    for (int first = 1; first < 10; first++) {
        for (int second = 1; second < first + 1; second++) {
            auto str = fmt::format("{1}*{0}={2:<2}", first, second, first * second);
            std::cout << str << " ";
        }
        std::cout << std::endl;
    }
}

void testabseil() {
    std::vector<std::string> v{ "foo", "bar", "baz" };
    std::string s = absl::StrJoin(v, "-");
    fmt::print("{0}\n", s);
}

//void testOpenCV() {
//    cv::utils::logging::setLogLevel(cv::utils::logging::LOG_LEVEL_SILENT);
//    cv::Mat myMat = cv::imread("C:/Users/FengisZZZ/Pictures/blog/head.jpg");
//    std::string s = fmt::format("{0} is {1}", "abra", 12);
//    std::cout << s << std::endl;
//    fmt::print("Elapsed time: {0:.2f} seconds\n", 1.2783);
//    int height = myMat.cols;
//    int width = myMat.rows;
//    fmt::print("the width:{0},the height:{1}\n", width, height);
//    cv::imshow("头像", myMat);
//    cv::waitKey();
//}

int main(int argc, char const* argv[]) {

    std::string s = fmt::format("{0} is {1}", "abra", 12);
    std::cout << s << std::endl;
    fmt::print("时间: {0:.2f} 秒\n", 1.2783);
    fmt::print("------------------------------------------------\n");
    testabseil();
    formatxx();
    // testOpenCV();
}
```



#### 4.1 必须

- **VScode setting.json**

  ```json
  "cmake.configureSettings": {
      "CMAKE_TOOLCHAIN_FILE": "C:/src/vcpkg/scripts/buildsystems/vcpkg.cmake"
  },
  ```

  

- **CMakeLists.txt**

  ```cmake
  cmake_minimum_required(VERSION 3.0.0)
  project(vcpkg_cmake VERSION 0.1.0)
  
  set(CMAKE_CXX_STANDARD 17)
  # 添加编译选项，源代码为utf-8选项，中文不乱码
  add_compile_options(/source-charset:utf-8)
  
  find_package(fmt CONFIG REQUIRED)
  # find_package(OpenCV CONFIG REQUIRED)
  find_package(absl CONFIG REQUIRED)
  
  add_executable(vcpkg_cmake main.cpp)
  
  target_link_libraries(${PROJECT_NAME} PRIVATE fmt::fmt)
  # target_link_libraries(${PROJECT_NAME} PRIVATE ${OpenCV_LIBRARIES})
  target_link_libraries(${PROJECT_NAME} PRIVATE absl::any absl::base absl::bits absl::city)
  ```

  **将 vcpkg 作为一个子模块**
  
  当您希望将vcpkg作为一个子模块加入到您的工程中时， 您可以在第一个 `project()` 调用之前将以下内容添加到 CMakeLists.txt 中， 而无需将 `CMAKE_TOOLCHAIN_FILE` 传递给cmake调用。
  
  ```
   set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake"
     CACHE STRING "Vcpkg toolchain file")
  ```
  
  使用此种方式可无需设置 `CMAKE_TOOLCHAIN_FILE` 即可使用vcpkg，且更容易完成配置工作。
  
  这种方法可能存在问题，建议直接设置Cmake Path
  
  
  
- `Clion中`， 在`CMake Setting`中设置 ，在 `CMake options`添加一行, 
  
  ```
  -DCMAKE_TOOLCHAIN_FILE=[vcpkg root]/scripts/buildsystems/vcpkg.cmake
  ```
  
  每个项目都要添加

#### 4.2 优化

- setting.json(全局)

  - 使用MSVC的工具链，但是生成Ninja更加简洁,生成compile_commands.json，为clangd插件提供补全
  - CMAKE_TOOLCHAIN_FILE 可以加载Vcpkg的包
  - clangd插件使用插件自己安装的clangd，同时提供了某些头文件

  `Ninja`使用预发布的版本在使用会有警告，可以选择github下载自己编译[Ninja github page](https://github.com/ninja-build/ninja)

  ```json
  "cmake.configureSettings": {
      "CMAKE_TOOLCHAIN_FILE": "C:/src/vcpkg/scripts/buildsystems/vcpkg.cmake"
  },
  "cmake.generator":"Ninja",
  "clangd.path": "c:\\Users\\f7ky\\AppData\\Roaming\\Code\\User\\globalStorage\\llvm-vs-code-extensions.vscode-clangd\\install\\14.0.3\\clangd_14.0.3\\bin\\clangd.exe"
  ```
  
- setting.json (工作区) 

  - clangd配置；
  - 在使用cmake插件debug时将结果输出在终端中；
  - 关闭C/C++语言功能，减少冲突；

  ```json
  {
      "clangd.arguments": [
          // 在后台自动分析文件（基于complie_commands)
          "--background-index",
          //cmake生成json位置
          "--compile-commands-dir=${workspaceFold}$/build",
          // clangd在创建索引使用的线程数
          "-j=6",
          // 启用 Clang-Tidy 以提供静态检查
          "--clang-tidy",
          // 建议风格：重载函数只会给出一个建议；反可以设置为detailed
          "--completion-style=bundled",
          // 启用,补全函数时，将会给参数提供占位符
          "--function-arg-placeholders=false",
          // 是否添加头文件
          "--header-insertion=never",
          // 输入建议中，已包含头文件的项与还未包含头文件的项会以圆点加以区分
          // "--header-insertion-decorators",
          // 让 Clangd 生成更详细的日志
          "--log=verbose",
          // pch存储在内存中，内存消耗更大，但是性能更优
          "--pch-storage=memory",
          // 输出的 JSON 文件更美观
          "--pretty",
          // 设置项目，或者用户config
          "--enable-config",
      ],
      // cmake-tools图形化 debug 结果输出在integratedTerminal
      // 因为只有使用MSVC工具链才有"console"选项，使用Mingw-64可能是"integratedTerminal",与launch.json相同
      "cmake.debugConfig": {
          "console":"integratedTerminal",
          "presentation": {
              "reveal": "always",
              "panel":"new",
              "close": true,
              "clear": true
          },
      },
      "C_Cpp.intelliSenseEngine": "Disabled"
  }
  ```

- launch.json

  - 快捷键调试的参数

  ```json
  {
      "version": "0.2.0",
      "configurations": [
          {
              "name": "MSVC",
              "type": "cppvsdbg",
              "request": "launch",
              "program": "${command:cmake.launchTargetPath}",
              "args": [],
              "stopAtEntry": false,
              "cwd": "${workspaceFolder}",
              "environment": [],
              "console": "integratedTerminal",
               "presentation": {
                  "reveal": "always",
                  "panel": "new",
                  "close": true,
                  "clear": true,
              },
          }
      ]
  }
  ```
  
- tasks.json

  ![tasks](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202207272052066.gif)


### 5. VScode使用MSVC单文件编译

**正常使用需要从VS命令行窗口打开**

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202206302219889.png" alt="image-20220629141805160" style="zoom: 40%;" />

观察到从命令实际上上启动了一个批处理命令，相当于配置了MSVC的环境变量,图中的命令需要使用` "type": "shell"`

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220629142144595.png" alt="image-20220629142144595" style="zoom: 40%;" />

`"type": "cppbuild"`需要此目录下的

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220630220746980.png" alt="image-20220630220746980" style="zoom: 67%;" />

所以我们只需要tasks.json执行这个命令

#### 5.1 tasks.json

`vcvarsall.bat`相当于是配置环境变量；`x64`生成64位的可执行文件；添加位置是指定中间文件的输出位置

![image-20220629144006206](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220629144006206.png)

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "Compile",
            "command": "\"C:\\Program Files\\Microsoft Visual Studio\\2022\\Community\\VC\\Auxiliary\\Build\\vcvarsall.bat\" x64 && cl.exe",
            "args": [
                "/std:c++20",
                "/Zi",
                "/EHsc",
                "/nologo",
                "/O2",
                "/W1",
                "/source-charset:utf-8",
                "/Fd${workspaceFolder}\\bin\\",
                "/Fo${workspaceFolder}\\bin\\${fileBasenameNoExtension}.obj",
                "/Fe${workspaceFolder}\\bin\\${fileBasenameNoExtension}.exe",
                "${file}",
                "/openmp:llvm"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": [
                "$msCompile"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
			"presentation": {
                "reveal": "always",
                "panel":"shared",
                "close": true,
                "clear": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```

也可以直接指定规定的模式,不需要指定x64

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220629144457736.png" alt="image-20220629144457736" style="zoom: 80%;" />

`args`是编译选项，在[官方网站](https://docs.microsoft.com/zh-cn/cpp/build/reference/align-section-alignment?view=msvc-170)，可以查看每个编译选项的含义，中文版的，非常nice

```
// c++标准是c++20
"/std:c++20",

// 保留完整的调试信息，不加的化没办法打断点
"/Zi",

// 标准 c + + 异常处理模型的完全编译器
"/EHsc",

// 在编译器启动时禁止显示版权横幅，在编译过程中显示信息性消息
"/nologo",

// 编译器选项是一次设置多个特定优化选项的一种快速方法
"/O2",

// W1 显示等级 1 (zhi'xian严重) 警告。 /W1 是命令行编译器中的默认设置。
"/W1",


// 设置编码
"/source-charset:utf-8",

// 没有 /Fd，PDB 文件名默认为 VCx0.pdb，其中 x 是当前Visual C++版本,指定输出位置
"/Fd${workspaceFolder}\\bin\\",

// 为 CL 编译器命令生成的所有对象文件设置输出目录
"/Fo${workspaceFolder}\\bin\\${fileBasenameNoExtension}.obj",

// 指定编译器创建的 .exe或 DLL 的名称和目录
"/Fe${workspaceFolder}\\bin\\${fileBasenameNoExtension}.exe",

"${file}",

```



#### 5.2 launch.json

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(Windows) DEBUG",
            "type": "cppvsdbg",
            "request": "launch",
            "program": "${workspaceFolder}\\bin\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "internalConsoleOptions": "neverOpen",
            "console": "integratedTerminal",
            "preLaunchTask": "Compile"
        }
    ]
}
```

#### 5.3  setting.json

在使用单文件编译时，如果禁用`c/c++`语言插件功能,在编译错误时提示编译错误，但是会自动运行上一次正确的编译结果，因此不能禁用

```json
// 不能直接禁用
// "C_Cpp.intelliSenseEngine": "Disabled",

"C_Cpp.autocomplete": "Disabled",
"C_Cpp.errorSquiggles": "Disabled",
"clangd.arguments": [
    // 在后台自动分析文件（基于complie_commands)
    "--background-index",
    //cmake生成json位置
    "--compile-commands-dir=${workspaceFold}$/build",
    // clangd在创建索引使用的线程数
    "-j=6",
    // 启用 Clang-Tidy 以提供静态检查
    "--clang-tidy",
    // 建议风格：重载函数只会给出一个建议；反可以设置为detailed
    "--completion-style=bundled",
    // 启用,补全函数时，将会给参数提供占位符
    "--function-arg-placeholders=false",
    // 是否添加头文件
    "--header-insertion=never",
    // 输入建议中，已包含头文件的项与还未包含头文件的项会以圆点加以区分
    // "--header-insertion-decorators",
    // 让 Clangd 生成更详细的日志
    "--log=verbose",
    // pch存储在内存中，内存消耗更大，但是性能更优
    "--pch-storage=memory",
    // 输出的 JSON 文件更美观
    "--pretty",
    // 设置项目，或者用户config
    "--enable-config",
],
```

