# 安装cmake

linux下直接使用包管理工具安装

```bash
apt update
sudo apt install cmake
```

如果默认安装的版本太低，可以手动去官网下载

![image-20240312215208467](cmake.assets/image-20240312215208467.png)

下载.sh文件和压缩包都行

.sh运行或者.tar解压之后，得到cmake-3.27.9-linux-x86_64

```bash
cd cmake-3.27.9-linux-x86_64
cd bin
./cmake --version

apt-get remove cmake	# 卸载之前安装的cmake
# 软连接 注意是绝对路径 否则会出错
ln -s /home/cmake-3.27.9-linux-x86_64/bin/cmake /usr/bin/cmake
cd /home
cmake --version	# 验证是否成功
```

# 构建项目

```bash
# CMake 2.x
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j4
make install
cd ..

# CMake 3.x
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel 4
cmake --build build --target install
```



# 常用命令

[使用CMake开发C++工程 | SIRLIS (gitee.io)](https://sirlis.gitee.io/posts/c-cmake-development/#1-引言)

## 版本与编译选项配置

```cmake
# 指定项目名称为 test
project(test)
# 指定 cmake 的最低版本
cmake_minimum_required(VERSION 3.10)
# 设置 cmake 的 C++ 版本为 C++11
set(CMAKE_CXX_STANDARD 11)

set(CMAKE_BUILD_TYPE Release)
```

## 可执行程序配置

```cmake
add_executable(${PROJECT_NAME} function.cpp main.cpp
    function.h include/a.h include/b.h 3rdparty/glm/rotate.h)
```

## 头文件搜索

为了避免繁琐的头文件列写过程，我们可以 `add_executable()` 后，指定其头文件搜索路径

```cmake
# 可以避免在每个源文件中手动指定头文件的完整路径

# 添加 include 目录为头文件搜索路径
target_include_directories(${PROJECT_NAME} PRIVATE include)
# 添加 include/glm 目录为头文件搜索路径
target_include_directories(${PROJECT_NAME} PRIVATE 3rdparty/glm)

```

## 自动导出头文件路径列表

```cmake
add_executable(${PROJECT_NAME} function.cpp main.cpp)
# 从根目录下开始遍历寻找所有头文件，存到 all_h 列表
file(GLOB_RECURSE all_h ./*.h)
foreach(HEADER ${all_h})
# 对于每一个头文件，得到其路径 h_path
get_filename_component(h_path ${HEADER} DIRECTORY)
# 将路径添加到 h_dirs 列表 
list(APPEND h_dirs ${h_path})
endforeach()
# 对 h_dirs 列表去除重复项
list(REMOVE_DUPLICATES h_dirs)
# 将 h_dirs 添加项目 test 的头文件搜索路径
target_include_directories(${PROJECT_NAME} PRIVATE ${h_dirs})
```

## 源文件搜索

```cmake
file(GLOB SOURCES "${PROJECT_SOURCE_DIR}/scr/*.cpp")
add_executable(main ${SOURCES})
```

## 子项目

通过子项目路径下的 `CMakeLists.txt`（path1）文件联合根目录下的 `CMakeLists.txt`（main）进行配置

```cmake
####### CMakeLists.txt (main)
# 添加子项目路径 path1
add_subdirectory(path1)
add_executable(test function.cpp main.cpp function.h ${path1src})

####### CMakeLists.txt (path1)
set(path1src path1/add.cpp path1/add.h)


#  搜索 ./test 目录下的所有源文件,并将其存储在 DIR_TEST_ARMA 变量中。
# (通常是 .cpp 或 .cc 文件)
aux_source_directory(./test DIR_TEST_ARMA)
```

## 查找包

```cmake
find_package(benchmark REQUIRED)
```

在 Linux 下，`find_package(benchmark REQUIRED)` 命令主要完成以下几个步骤:

1. 搜索系统中是否安装了 benchmark 库:
   - CMake 会在系统默认的库搜索路径（如 `/usr/lib`、`/usr/local/lib`）中查找 benchmark 库的头文件和链接库。
   - 可以通过设置环境变量 `CMAKE_PREFIX_PATH` 来指定额外的搜索路径。
2. 加载 benchmark 库的配置文件:
   - 如果系统中安装了 benchmark 库，CMake 会尝试加载 benchmark 的配置文件(通常是 `benchmark-config.cmake` 或 `benchmark-targets.cmake`)。
   - 这些配置文件包含了 benchmark 库的详细信息,如头文件路径、链接库路径、版本号等。
3. 检查 benchmark 库的版本是否满足要求:
   - 如果在前一步中找到了 benchmark 的配置文件,CMake 会检查 benchmark 库的版本是否满足当前项目的要求。
   - 如果使用了 `REQUIRED` 关键字,则必须找到满足版本要求的 benchmark 库,否则 CMake 配置过程会失败。
4. 将 benchmark 库的信息添加到当前项目的构建系统中:
   - 如果找到了满足要求的 benchmark 库,CMake 会将其头文件路径、链接库路径等信息添加到当前项目的构建系统中。
   - 这样在后续的源代码编译和链接过程中,就可以正确地使用 benchmark 库。

CMake 在搜索 `benchmark-config.cmake` 文件时,会依次查找以下几个位置:

1. `/lib/cmake/benchmark/`
2. `/lib/benchmark/cmake/`
3. `/share/cmake/benchmark/`
4. `/share/benchmark/cmake/`

其中 `` 表示 benchmark 库的安装路径,可能是系统默认路径(如 `/usr` 或 `/usr/local`)或者自定义安装路径。

例如：

```bash
root@ba0a3e4c230d:/usr# find ./ -name benchmark.h
./local/include/benchmark/benchmark.h
root@ba0a3e4c230d:/usr# find ./ -name libbench*.a
./local/lib/libbenchmark_main.a
./local/lib/libbenchmark.a
root@ba0a3e4c230d:/usr# find ./ -name bench*.cmake
./local/lib/cmake/benchmark/benchmarkTargets-release.cmake
./local/lib/cmake/benchmark/benchmarkConfigVersion.cmake
./local/lib/cmake/benchmark/benchmarkConfig.cmake
./local/lib/cmake/benchmark/benchmarkTargets.cmake
```

## 测试

`enable_testing()`启用 CMake 的测试功能。

在 CMakeLists.txt 文件中添加了 `enable_testing()` 之后,你可以通过以下步骤来使用 CMake 的测试功能:

1. **编写测试用例**:

   - 在 `./test` 目录下编写单元测试文件,使用 Google Test 框架编写测试用例。
   - 确保测试文件与项目源文件中的模块或功能对应。

2. **注册测试**:

   - 在 CMakeLists.txt 文件中,添加 `add_test` 命令来注册测试用例。

   - 示例:

     ```cmake
     add_test(NAME my_test COMMAND my_test_exe  --gtest_output=xml:test_results.xml)
     ```

     这会创建一个名为my_test的测试，并在运行`kuiper_datawhale_course2`可执行文件时加上 `--gtest_output=xml:test_results.xml`参数。

3. **构建并运行测试**:

   - 使用 CMake 构建项目:

     ```bash
     mkdir build && cd build
     cmake ..
     cmake --build .
     ```

   - 运行测试:

     ```
     ctest
     ```

     这会自动运行所有注册的测试用例,并显示测试结果。

4. **分析测试结果**:

   - 测试结果会输出到控制台,并生成 `test_results.xml` 文件。
   - 可以使用 Google Test 提供的工具或第三方工具(如 Jenkins、Travis CI 等)来分析测试结果。