## CMake 的基础用法

```cmake
# 需要的最低的CMake版本
cmake_minimum_required(VERSION 3.19)

# 设置C++编译的版本，如11,14,17...
set(CMAKE_CXX_STANDARD 11)

# 设置项目的名称
project(hello)

# 指定生成的文件名和执行文件
add_executable(main main.cpp)

# 添加编译选项,多个选项之间用空格隔开
add_compile_options(-g -Wunused)

# 根据编译器类型设置不同的编译选项
# if(MSVC)
#     # MSVC编译器选项
#     add_compile_options(/W4)
# else()
#     # GCC/Clang编译器选项
#     add_compile_options(-g -Wunused)
# endif()

# 给指定的目标添加编译选项
# target_compile_options(main PUBLIC -Wall -Werror)
```



#### 多文件编译：

**单个添加：**

```cmake
cmake_minimum_required(VERSION 3.19)

project(hello)

# file:获取src目录下的所有.cpp文件，并将其保存在变量中
file(GLOB_RECURSE SOURCES "src/*.cpp")

# include_directories: 头文件搜索路径
include_directories(./)

#add_executable(main src/sub.cpp src/add.cpp main.cpp)
add_executable(main ${SOURCES} main.cpp)
```



**问题一：**

如果源文件分布在多个目录下怎么办？

```cmake
file(GLOB_RECURSE SOURCES "src/*.cpp" "src1/*.cpp")
```



**问题二：**

如果头文件分布在多个目录下怎么办？

```cmake
# 依次往()里面写，如./ ./include1 ./include2
include_directories(./ ./include)
```



#### 多目标编译：

```cmake
add_executable(main ${SOURCES} main.cpp)
add_executable(test ${SOURCES} test.cpp)
```



#### 静态库编译 & 链接



**编译：**

```cmake
set(MUL_SOURCES ./mul/mul.cpp)
# add_library 指定从某些源文件创建库文件（静态库和动态库）
add_library(mul STATIC ${MUL_SOURCES})

# 以上写法还可以写成下面的方法
target_link_directories(main PUBLIC ./build/Debug)
target_link_libraries(main mul) # 只会链接到main这个文件里面

#注意：windows下静态库文件以.lib结尾，Linux下以.a结尾
```

**链接：**

```cmake
link_directories(./build/Debug)
link_libraries(mul)
```



#### 动态库编译 & 链接

**编译：**

```cmake
set(MUL_SOURCES ./mul/mul.cpp)
add_library(mul SHARED ${MUL_SOURCES})

# 以上写法还可以写成下面的方法
target_link_directories(main PUBLIC ./build/Debug)
target_link_libraries(main mul) # 只会链接到main这个文件里面
```

**链接：**

```cmake
link_directories(./build/Debug)
link_libraries(mul)
```

