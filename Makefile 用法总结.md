## Makefile 用法总结

1. ### **编译一个简单的程序：**

   ```cpp
   // 使用gcc编译
   // 方式一：两步，中间生成一个-o文件
   // (1). gcc -c main.cpp -o main.o
           // 此时生成main.o文件
   // (2). gcc main.o -o main -lstdc++
           // 此时生成main.exe文件
   // 方式二：一步，中间不再生成main.o文件，直接生成main.exe文件
   // (1). gcc main.cpp -o main -lstdc++
   
   // 使用g++编译,不用包含标准库了
   // 方式一：两步，中间生成一个-o文件
   // (1). g++ -c main.cpp -o main.o
           // 此时生成main.o文件
   // (2). g++ main.o -o main 
           // 此时生成main.exe文件
   // 方式二：一步，中间不再生成main.o文件，直接生成main.exe文件
   // (1). g++ main.cpp -o main
   
   ```

   

2. ### **编译流程：**

   ![image-20250329172354983](C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250329172354983.png)

   ####   方式一：

   （1）预处理：生成main.i文件

   ​			g++ -E main.cpp -o main.i

     (2）编译：    生成main.s文件

   ​			g++ -S main.i -o main.s

​	 (3）汇编：   生成main.o文件

​				g++ -c main.s -o main.o

​	 (4）链接：   生成main文件

​				g++ main.o -o main

#### 	方式二：

​                将预处理，编译和汇编合在一起使用：

​		g++ -c main.cpp -o main.o

​        	然后在使用：

​		g++ main.o -o main

#### 	方式三：

​        	或者一步到位使用：

​		g++ main.cpp -o main

3. ### **编译错误 & 链接错误：**

   ​	记得包含.h文件（.h声明，.cpp实现）

   ​	编译两个文件（一步到位）：

   ​	g++ add.cpp main.cpp -o main

   ​	注意在源文件里面要包含对应的头文件（如果不包含的话，那么就会对源文件就缺少编译环节，可能出错，要养成有良好的习惯！）

4. ### 头文件搜索路径：

   <> :绝对路径   ，在编译时候可以使用-I来告诉编译器在当前目录下来寻找，例如g++ -I. add.cpp main.cpp -o main

​	"" : 相对路径（相对于源文件）

5. ### 多文件编译：

   （1）.可以对每一个源文件进行编译，然后再对生成的所有目标文件进行链接

   （2）.可以直接对所有的文件统一进行编译生成目标文件

   ​           e.g：g++ sub.cpp add. cpp main.cpp -o main

6. ### makefile 基础用法：

   多文件文件编译，采用make来优化编译

   新建Makefile文件

   在里面输入：

   ```Makefile
   // 写法一：
   // g++前面是一个tab键，而不是空格
   target:
   	g++ -I. add.cpp sub.cpp main.cpp -o main
   // Makefile里如果有多个目标，可以通过make加目标名称来执行
   // 指定的目标，如果不指定目标，则默认选择第一个目标执行
           
   // 写法二：
   target: main
   main:
   	g++ _I. add.cpp sub.cpp main.cpp -o main
   // 写法三：(错误，因为target中先编译了main,而main中add.o等文件还没有生成)
   target: main add_o sub_o main_o
   main: 
   	g++ add.o sub.o main.o main
   add_o:
   	g++ -I. -c add.cpp -o add.o
   sub_o:
   	g++ -I. -c sub.cpp -o sub.o
   main_o:
   	g++ -I. -c main.cpp -o main
   // 修改：
   target: add_o sub_o main_o main
   main: 
   	g++ add.o sub.o main.o main
   add_o:
   	g++ -I. -c add.cpp -o add.o
   sub_o:
   	g++ -I. -c sub.cpp -o sub.o
   main_o:
   	g++ -I. -c main.cpp -o main        
   ```

   

7. ### Makefile 升级用法:

   ```makefile
   // 对于写法三，可以进一步进行优化
   target: add.o sub.o main.o main
   
   main:
   	g++ add.o sub.o main.o -o main
   %.o: %.cpp
   	g++ -I. -c $< -o $@
   // 所有.o文件都是由相同名称的.cpp文件编译生成
   // '<' 和 '@' 称为 "自动化变量"
   // < : 表示所有依赖的挨个值
   // @ : 表示所有目标的挨个值
   
   // 进一步优化：
   OBJS = add.o sub.o main.o
   
   target:$(OBJS) main
   
   main:
   	g++ $(OBJS) -o main
   %.o: %.cpp
   	g++ -I. -c $< -o $@
   // 说明：
   // 引用变量时，加下小括号或者大括号，这样可以更加安全的引用变量
   // = 是最基本的赋值
   // := 是覆盖之前的值
   // ?= 是如果没有被赋值过就赋予等号后面的值
   // += 是添加等号后面的值
   
   // clean:
   	rm -rf $(OBJS) main
   // 这样可以使得修改后再次编译文件时将原本生成的.o文件和main文件都删除
   ```

8. #### Makefile 高级用法：

   ```makefile
   # 找出当前目录下，所有的源文件(以.cpp结尾)
   SRCS := $(shell find ./* -type f | grep "\.cpp")
   $(warning SRCS is (SRCS))
   
   # 确定cpp源文件对应的目标文件
   OBJS := $(patsubst %.cpp,%.o, $(filter %.cpp,$(SRCS)))
   $(warning OBJS is (OBJS))
   
   # 编译选项
   CFLAGS := -g -O2 -Wall -Werror -Wno-unused -ldl -std=c++11
   
   # 搜索路径
   INCLUDE := -I.
   
   MAIN_SRC := hello.cpp
   MAIN_OBJ := $(MAIN_SRC: %.cpp=%.o)
   MAIN_EXE := main
   
   TEST_SRC := test.cpp
   TEST_OBJ := $(TEST_SRC: %.cpp=%.o)
   TEST_EXE := test
   
   target: $(MAIN_EXE) $(TEST_EXE)
   
   $(MAIN_EXE): $(OBJS) $(MAIN_OBJ)
   	g++ -o $@ $^ $(INCLUDE) $(CFLAGS)
   	
   $(TEST_EXE): $(OBJS) $(TEST_OBJ)
   	g++ -o $@ $^ $(INCLUDE) $(CFLAGS)
   	
   %.o: %.cpp
   	g++ $(INCLUDE) $(CFLAGS) -c $< -o $@
   
   clean:
   	rm -rf $(OBJS) main
   ```

9. #### Makefile 多路口编译：

   ```makefile
   # 多路口编译文件
   # 编译test.cpp 和 helllo.cpp
   # 找出当前目录下，所有的源文件(以.cpp结尾)
   SRCS := $(shell find ./* -type f | grep "\.cpp" | grep -v "./hello\.cpp" | grep -v "./test\.cpp")
   $(warning SRCS is (SRCS))
   
   # 确定cpp源文件对应的目标文件
   OBJS := $(patsubst %.cpp,%.o, $(filter %.cpp,$(SRCS)))
   $(warning OBJS is (OBJS))
   
   # 编译选项
   CFLAGS := -g -O2 -Wall -Werror -Wno-unused -ldl -std=c++11
   
   # 搜索路径
   INCLUDE := -I.
   
   MAIN_SRC := hello.cpp
   MAIN_OBJ := $(MAIN_SRC: %.cpp=%.o)
   MAIN_EXE := main
   
   TEST_SRC := test.cpp
   TEST_OBJ := $(TEST_SRC: %.cpp=%.o)
   TEST_EXE := test
   
   target: $(MAIN_EXE) $(TEST_EXE)
   
   $(MAIN_EXE): $(OBJS) $(MAIN_OBJ)
   	g++ -o $@ $^ $(INCLUDE) $(CFLAGS)
   	
   $(TEST_EXE): $(OBJS) $(TEST_OBJ)
   	g++ -o $@ $^ $(INCLUDE) $(CFLAGS)
   	
   %.o: %.cpp
   	g++ $(INCLUDE) $(CFLAGS) -c $< -o $@
   
   clean:
   	rm -rf $(OBJS) main
   ```

   ```makefile
   # 单路口编译文件通用模板
   # e.g.编译hello.cpp
   
   #指定编译器
   CC := g++
   
   # 找出当前目录下，所有的源文件(以.cpp结尾)
   SRCS := $(shell find ./* -type f | grep "\.cpp" | grep -v "./hello\.cpp")
   $(warning SRCS is (SRCS))
   
   # 确定cpp源文件对应的目标文件
   OBJS := $(patsubst %.cpp,%.o, $(filter %.cpp,$(SRCS)))
   $(warning OBJS is (OBJS))
   
   # 编译选项
   CFLAGS := -g -O2 -Wall -Werror -Wno-unused -ldl -std=c++11
   
   # 搜索路径
   INCLUDE := -I.
   
   MAIN_SRC := hello.cpp
   MAIN_OBJ := $(MAIN_SRC: %.cpp=%.o)
   MAIN_EXE := main
   
   target: $(MAIN_EXE) 
   
   $(MAIN_EXE): $(OBJS) $(MAIN_OBJ)
   	$(CC) -o $@ $^ $(INCLUDE) $(CFLAGS)
   %.o: %.cpp
   	$(CC) $(INCLUDE) $(CFLAGS) -c $< -o $@
   
   clean:
   	rm -rf $(OBJS) main
   ```

   **注意事项：**

  	（1）将add.cpp，sub.cpp挪到src目录下仍然可以正常编译

​	  （2）将add.h，sub.h挪到include目录下只需修改INCLUDE既可编译（在INCLUDE 后添加-I./include）

10. #### 静态库编译：

    ```makefile
    MUL_SRC := ./lib/mul.cpp
    MUL_OBJ := $(MUL_SRC:%.cpp=%.o)
    MUL_LIB := ./lib/libmul.a
    
    static : $(MUL_LIB)
    
    $(MUL_LIB): $(MUL_OBJ)
    	ar rs $@ $^
    	
    # 编译静态库的指令
    make static
    
    # 静态链链接
    LDFLAGS := -L./lib -lmul
    
    # 编译target中加上$(LDFLAGS),即
    $(MAIN_EXE): $(OBJS) $(MAIN_OBJ)
    	$(CC) -o $@ $^ $(INCLUDE) $(CFLAGS) $(LDFLAGS)
    	
    # 动态链接库
    MUL_DLL := ./lib/libmul.so
    
    shared: $(MUL_DLL)
    $(MUL_DLL): $(MUL_OBJ)
    	$(CC) -o $@ $^ $(INCLUDE) $(CFLAGS) -fPIC -shared
    	
    ```

    **注意事项：为了编译器能找到动态库，需要完成下面两种方式之一**

    （1）将动态库文件拷贝到/usr/local/lib目录下，程序运行时加载动态库会到该目录下查找（永久）

    （2）通过环境变量LD_LIBRARY_PATH来指定动态库查找目录(临时)

    ​	  export LD_LIBRARY_PATH=./lib

11. #### 动态库编译：
