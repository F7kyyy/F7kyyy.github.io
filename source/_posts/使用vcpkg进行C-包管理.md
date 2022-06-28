---
title: 使用vcpkg进行C++包管理
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
>主要是还在考试的过程中，完全不想准备考研，在知乎刷到了一篇关于现代c++的介绍，介绍了一些流行的第三方库，如google 的 `abseil`,facebook 的`folly`,所以想安装试一试。一般都需要编译安装，非常的麻烦，所以考虑使用msys2来安装，但是`folly`库msys2居然都没有,果然Windows 还是老老实实的用MSVC吧:dog:。Vcpkg经过几年的发展已经完全可用了，开源库都有Vcpkg的安装方式。



### 1. 安装Visual Studio 2022

我们可以选择只安装[MS Build Toos](https://visualstudio.microsoft.com/zh-hans/downloads/),但是当我们选择最基本的C++开发组件，大概需要4G左右存储空间，语言包需要额外选上英文，这是Vcpkg依赖。

![image-20220627223858130](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627223858130.png)

不过，我们为什么不选择直接安装Visual Studio 2022 ，体验一下宇宙第一IDE，毕竟完整安装也只要8G左右空间。**强烈建议直接安装完整版，因为只安装开发工具的话想要再次完整安装会重新下载另外一份开发工具，十分扯淡**，同样的`语言包需要额外选上英文`

![image-20220627224405597](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627224405597.png)

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

![image-20220627225537803](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627225537803.png)

![image-20220627225902749](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627225902749.png)

### 3.集成在VS中

#### 3.1 全局集成

执行下命令，直接新建一个VS项目，即可使用。

```powershell
vcpkg integrate install
```

![image-20220627231533379](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627231533379.png)

#### 3.2 工程集成和CMake集成

详见github

### 4. 在Visual Studio Code中使用

这个项目是同时使用`OpenCV`,`fmt`库

![image-20220627233132298](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/image-20220627233132298.png)

#### 4.1 必须

- **VScode setting.json**

  ```json
  "cmake.configureSettings": {
      "CMAKE_TOOLCHAIN_FILE": "C:/src/vcpkg/scripts/buildsystems/vcpkg.cmake"
  },
  ```

  

- **CMakeLists.txt**

  ```cmake
  cmake_minimum_required(VERSION 3.10)
  project(TestOpenCV)
  
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
  
  find_package(OpenCV REQUIRED)
  find_package(fmt REQUIRED)
  
  add_executable(TestOpenCV main.cpp )
  target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBRARIES})
  target_link_libraries(${PROJECT_NAME} fmt::fmt)
  ```

  **将 vcpkg 作为一个子模块**

  当您希望将vcpkg作为一个子模块加入到您的工程中时， 您可以在第一个 `project()` 调用之前将以下内容添加到 CMakeLists.txt 中， 而无需将 `CMAKE_TOOLCHAIN_FILE` 传递给cmake调用。

  ```
   set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake"
     CACHE STRING "Vcpkg toolchain file")
  ```

  使用此种方式可无需设置 `CMAKE_TOOLCHAIN_FILE` 即可使用vcpkg，且更容易完成配置工作。
  
  这种方法可能存在问题，建议直接设置Cmake Path
  
  `Clion中`， 在`CMake Setting`中设置 ，在 `CMake options`添加一行, 
  
  ```
  -DCMAKE_TOOLCHAIN_FILE=[vcpkg root]/scripts/buildsystems/vcpkg.cmake
  ```
  
  每个项目都要添加

#### 4.2 优化、、、、、、、、、、l

- setting.json(全局)

  - 使用MSVC的工具链，但是生成Ninja更加简洁,为ckangd插件提供补全
  - CMAKE_TOOLCHAIN_FILE 可以加载Vcpkg的包
  - clangd插件使用自己安装的clangd

  ```json
  "cmake.configureSettings": {
      "CMAKE_TOOLCHAIN_FILE": "C:/src/vcpkg/scripts/buildsystems/vcpkg.cmake"
  },
  "cmake.generator": "Ninja",
  "clangd.path": "c:\\Users\\f7ky\\AppData\\Roaming\\Code\\User\\globalStorage\\llvm-vs-code-extensions.vscode-clangd\\install\\14.0.3\\clangd_14.0.3\\bin\\clangd.exe"
  ```

- setting.json(工作区)

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
      "cmake.debugConfig": {
          "console":"integratedTerminal"
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
              "console": "integratedTerminal"
          }
      ]
  }
  ```

  