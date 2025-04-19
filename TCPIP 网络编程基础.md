## TCP/IP 网络编程基础



#### 一. 网络协议：TCP/IP

1. 应用层：主要负责为用户提供网络服务。
2. **传输层：**主要负责在网络中建立端到端的连接，提供可靠的数据传输。
3. 网络层：主要负责网络地址的分配和路由选择。
4. 数据链路层：主要负责传输数据帧。



#### 二. 进程间通信：IPC

 （1 - 5 都是同一个主机进程之间的通信）

1. 管道：pipe,父子进程之间的通信，ps -ef | grep nginx 
2. 信号：signal，kill -9 pid
3. 信号量：semaphore，多线程环境中
4. 消息队列：message，操作系统内核提供的，不是应用级别的kfaka，rabbitMQ
5. 共享内存：share memory，进程间通信效率最高
6. 套接字：**socket**，不同主机进程之间的通信



#### 三. 套接字：socket

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250407165827048.png" alt="image-20250407165827048" style="zoom:50%;" />

​                                                        <img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250407170115045.png" alt="image-20250407170115045" style="zoom:50%;" />   



1. 服务端和客户端初始化Socket，得到文件描述符
2. 服务端调用bind，绑定IP和端口
3. 服务端调用listen，进行监听
4. 服务端调用accept，等待客户端连接
5. 客户端调用connect，向服务端发起连接请求（TCP三次握手）
6. 服务端调用accept返回用于传输的Socket的文件描述符（和第一点得到的Socket不同）
7. 客户端使用write写入数据，服务端调用read读取数据
8. 客户端断开连接时会调用close，服务端也会调用close（TCP四次挥手）



#### 基本函数：

``` cpp
// 1. socket() - 创建套接字
int socket(int domain, int type, int protocol);
domain: 协议族，如 AF_INET(IPv4) 或 AF_INET6(IPv6)
type: 套接字类型，如 SOCK_STREAM(TCP) 或 SOCK_DGRAM(UDP)
protocol: 通常为0，表示默认协议
// socket()返回一个整数，若为-1表示创建失败
    
// 2. bind() - 绑定地址和端口
int bind(int sockfd, const struct sockaddr *addr,socklen_t addrlen);
socked: socket() 返回的套接字描述符
addr: 指向要绑定的地址结构指针
addrlen: 地址结构长度
    
// 3. listen() - 开始监听连接
int listen(int sockfd, int backlog);
sockfd: 已绑定的套接字描述符
backlog: 等待连接队列的最大长度
    
// 4. accept() - 接受连接
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
sockfd: 监听中的套接字描述符
addr: 用于存储客户端地址信息的结构体指针
addrlen: 地址结构长度指针
    
// 5. recv() - 接收数据
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
sockfd: 已连接的套接字描述符
buf: 接收数据的缓冲区
len: 缓冲区长度
flags: 通常为0
    
// 6. send() - 发送数据
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
sockfd: 已连接的套接字描述符
buf: 要发送的数据缓冲区
len: 数据长度
flags: 通常为0

// 7. close() - 关闭套接字
int close(int fd);

// 8. connect() - 客户端连接服务端
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```



#### sockaddr_in 结构体初始化：

```cpp
// 代码解释
struct sockaddr_in sockaddr;  // sockaddr_in(Socket Address Internet), 声明一个sockaddr_in结构体变量，用于存储IPv4地址和端口信息

std::memset(&sockaddr, 0, sizeof(sockaddr)); // 将整个结构体清零

sockaddr.sin_family = AF_INET; // 这是 IPv4 套接字的固定设置,使用IPv4协议，AF_INET(Address Family Internet)

sockaddr.sin_addr.s_addr = inet_addr(ip.c_str()); // 将点分十进制的 IP 字符串转换为网络字节序的 32 位整数

sockaddr.sin_port = htons(port); // 设置端口号,htons() 函数将主机字节序的端口号转换为网络字节序(大端序),port 是主机字节序的端口号
```



#### 简单服务端：

```cpp
// server.cpp
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <cstring>
#include <string>
#include <unistd.h>
#include <arpa/inet.h>
using std::string;

int main() {
	// 1. 创建 socket
	int sockfd = ::socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (sockfd < 0) {
		printf("create socket error: errno = %d errmsg = %s",errno,strerror(errno));
		return 1;
	} else {
		printf("create socket success!");
		printf("\n");
	}

	// 2. 绑定 socket
	string ip = "127.0.0.1";
	int port = 8080;

	struct sockaddr_in sockaddr;
	std::memset(&sockaddr, 0, sizeof(sockaddr));
	sockaddr.sin_family = AF_INET;
	sockaddr.sin_addr.s_addr = inet_addr(ip.c_str());
	sockaddr.sin_port = htons(port);

	if (::bind(sockfd, (struct sockaddr *)&sockaddr, sizeof(sockaddr)) < 0) {
		printf("socket bind error: error = %d errmsg = %s\n", errno, strerror(errno));
		return 1;
	} else {
		printf("socket bind success: ip = %s port = %d\n", ip.c_str(), port);
	} 

	// 3. 监听 socket
	if (::listen(sockfd, 1024) < 0) {
		printf("socket listen error: errno = %d, errmsg = %s\n",errno, strerror(errno));
	} else {
		printf("socket listening...");
		printf("\n");
	}

	while (true) {
		// 4. 接受客户端的连接
		int connfd = ::accept(sockfd, nullptr, nullptr);
		if (connfd < 0) {
			printf("socket accept error: errno = %d errmsg = %s\n",errno, strerror(errno));
			return 1;
		} 

		char buf[1024] = {0};

		// 5. 接收客户端的数据
		size_t len = ::recv(connfd, buf, sizeof(buf), 0);
		printf("recv: connfd = %d msg = %s\n",connfd, buf);

		// 6. 向客户端发送数据
		::send(connfd, buf, len, 0);
	}

	7. 关闭 socket
	::close(sockfd);
	return 0;
}
```



#### 简单客户端：

```cpp
// client.cpp
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <cstring>
#include <string>
#include <unistd.h>
#include <arpa/inet.h>

using std::string;

int main() {
	// 1. 创建 socket
	int sockfd = ::socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (sockfd < 0) {
		printf("create socket error: errno = %d errmsg = %s",errno,strerror(errno));
		return 1;
	} else {
		printf("create socket success!\n");
	}
	
	// 2. 连接服务端
	string ip = "127.0.0.1";
	int port = 8080;
	struct sockaddr_in sockaddr;
	std::memset(&sockaddr, 0, sizeof(sockaddr));
	sockaddr.sin_family = AF_INET;
	sockaddr.sin_addr.s_addr = inet_addr(ip.c_str());
	sockaddr.sin_port = htons(port);
	if (::connect(sockfd, (struct sockaddr *)&sockaddr, sizeof(sockaddr)) < 0) {
		printf("socket connect error: error = %d errmsg = %s\n", errno, strerror(errno));
		return 1;
	}

	// 3. 向服务端发送数据
	string data = "hello world";
	::send(sockfd, data.c_str(), data.size(), 0);

	// 4. 接收服务端的数据
	char buf[1024] = {0};
	::recv(sockfd, buf, sizeof(buf), 0);
	printf("recv: %s\n", buf);

	// 5. 关闭 socket
	::close(sockfd);
	return 0;
}
```



#### 四. Socket 封装



#### 五. Socket属性

​	**阻塞 IO：**

​	当用户线程发出IO请求之后，内核会查看数据是否就绪，如果没有就绪就会等待数据就绪，而用户线程就会处于阻塞状态，用户线程交出CPU。当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除阻塞状态。

​	

​	**非阻塞IO：**

​	当用户线程发起read操作后，并不需要等待，而是马上得到结果。如果结果是一个error时，它就知道有数据还没有准备好，于是它可以再次发起read操作。如果内核中的数据准备好了，它就将数据拷贝到用户线程。

​	在非阻塞IO模型中，用户线程需要不断地轮询内核数据是否就绪，也就是说非阻塞IO不会交出CPU，而会一直占用CPU。

​	

​	**发送缓冲区：**

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250408105300815.png" alt="image-20250408105300815" style="zoom:50%;" />

​	socket没法直接将数据发送到网卡，所以只能现将数据发送到操作系统数据发送缓冲区。然后网卡从数据发送缓冲区中获取数据，再发送到接收方。

​	如果用户程序发送数据的速度比网卡读取的速度快，那么发送缓冲区将会很快被填满，这个时候send会被阻塞，也就是写入发生阻塞。

​	

​	**接收缓冲区：**

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250408110333284.png" alt="image-20250408110333284" style="zoom:50%;" />

​	首先接收方机器网卡接受到发送放的数据后，先将数据保存到操作系统接收缓冲区。用户程序感知到操作系统缓冲区的数据后，主动调用接收数据的方法来获取数据。

​	如果数据接收缓冲区为空，这个时候recv会被阻塞，也就是读取发生阻塞。

​	注意：发送缓冲区和接受缓冲区这两个区域是每一个socket连接都有的。本质上而言，就是内核中的两块内存空间，socket创建完成后，这两块内存空间就开辟出来了。

​	

​	**SO_LINGER：**

​	设置函数close()关闭TCP连接时的行为。缺省close()的行为是，如果有数据残留在socket发送缓冲区中则系统将继续发送这些数据给对方，然后等待被确认，然后返回。

​	

​	**SO_KEEPALIVE：**

​	不论是服务端还是客户端，一方开启KeepAlive功能后，就会自动在规定时间内向对方发送心跳包，而另一方在接收到心跳包后就会立刻自动回复，以告诉对方我仍然在线。

​	

​	**SO_REUSEADDR：**

​	SO_REUSEADDR是一个很有用的选项，一般服务器的监听socket都应该打开它。它的大意是允许服务器bind一个地址，即使这个地址当前已经存在已建立的连接。



#### 六. IO 多路复用

​	在多路复用的IO模型中，会有一个专门的线程不断去轮询多个socket的状态，只有当socket真正有读写事件时，才真正调用实际的IO读写操作。IO多路复用的优势在于，可以处理大量并发的IO，而不用消耗太多CPU/内存。



三种常用的轮询方法：select，poll，epoll

