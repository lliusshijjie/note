#### **#Reactor**

1. ***定义***：一种事件驱动编程模型，用于高效处理并发I/O操作，常见于网络服务器

2. ***核心思想***：（1）.通过单线程或少量线程监听多个I/O事件（如连接请求、数据到达）。

   ​		   （2）.事件触发后分发给对应的处理器（Handler）异步处理。

   3.***优势***：高并发下资源消耗低（避免为每个连接创建线程）和 适用于C10K等高并发场景

   4.***典型应用***：Nginx、Node.js、Netty等网络框架的核心模型

   5.***流程示例***：Reactor 监听所有事件源（如Socket）；事件到达后，Reactor 将其分派给预注册的处理器；处理器非阻塞处理任务，或 提交到线程池执行



```C++
#include<bits/stdc++.h>

int main() {
    std::cout << "Hello World!" << std::endl;
    return 0;
}
```



```python
print("Hello World")
```



