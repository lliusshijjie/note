## C++ 多线程：线程池从设计到实现



#### 为什么要用线程池？

1. 进程之间切换代价比较高，线程之间切换代价比较小
2. 解决 CPU 和 IO 速度不匹配的问题，多线程更适合 IO 切换频繁的场景
3. 充分利用多核 CPU 资源，提高程序的并发效率



#### 目标：

    1. 实现一个基于 posix thread 的线程池
    1. 实现一个基于c++ 11 thread 的线程池
    1. 两个版本的线程池架构设计完全一样



#### 整体架构：

​						<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250331105622601.png" alt="image-20250331105622601" style="zoom:80%;" />



#### 实际应用：

1. 任务接口
2. 初始化线程池
3. 任务分发