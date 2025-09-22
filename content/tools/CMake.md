---
tags:
  - cmake
  - tool
created: 2023-03-16T00:43:34 (UTC +08:00)
---

# CMake

## 01-基础

### A-hello-cmake

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/28/lu18012y8s8_tmp_baf698b4f118340b-f18706.png)

一个非常简单的CMake项目，只包含两个文件：

1. CMakeLists.txt - 包含CMake命令的文件，表示你想怎么运行项目。在执行CMake命令时，CMake会首先查看这个文件，如果不存在会报错
   
1. main.cpp - 简单的源代码
   

**CMakeLists** **参数说明**

1. cmake_minimum_required ：项目支持的最小的CMake版本

1. project (hello_cmake)：项目名称

1. add_executable(hello_cmake main.cpp)：add_executable()命令指定应从指定的源文件构建可执行文件，在本例中为main.cpp。add_executable()函数的第一个参数是要构建的可执行文件的名称，第二个参数是要编译的源文件列表。如图，项目构建后会创建hello_cmake 可执行文件<img src="https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/28/lu18012y8s8_tmp_d7acfd156c9d38b0-28bc9a.png" style="zoom:33%;" />|

  

```PowerShell
$ cmake ..
-- The C compiler identification is GNU 11.2.0
-- The CXX compiler identification is GNU 11.2.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/cjj/cmake-examples/01-basic/A-hello-cmake/build
$ make
[ 50%] Building CXX object CMakeFiles/hello_cmake.dir/main.cpp.o
[100%] Linking CXX executable hello_cmake
[100%] Built target hello_cmake
$ ./hello_cmake
Hello CMake!
```

  

## B-hello-headers

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/11/28/lu18012y8s8_tmp_7e50e32d6f5f8705-f1b156.png)

与hello_cmake不同，这个项目新增了头文件。

```PowerShell
$ tree
.
├── CMakeLists.txt
├── include
│   └── Hello.h
├── README.adoc
└── src
    ├── Hello.cpp
    └── main.cpp
```

  

| **Variable** | **Info** |
| ------------ | -------- |
|CMAKE_SOURCE_DIR|The root source directory|
|CMAKE_CURRENT_SOURCE_DIR|The current source directory if using sub-projects and directories.|
|PROJECT_SOURCE_DIR|The source directory of the current cmake project.|
|CMAKE_BINARY_DIR|The root binary / build directory. This is the directory where you ran the cmake command.|
|CMAKE_CURRENT_BINARY_DIR|The build directory you are currently in.|
|PROJECT_BINARY_DIR|The build directory for the current project.|


