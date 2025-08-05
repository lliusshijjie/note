# Docker 技术

## 原理

<img src="C:\Users\10424\AppData\Roaming\Typora\typora-user-images\image-20250804160252539.png" alt="image-20250804160252539" style="zoom:67%;" />



### 容器

1. **容器类似轻量级的VM**

2. **容器共享操作系统内核**

3. **容器拥有自己的文件系统，CPU，内存，进程空间**

4. **容器之间相互隔离，不会互相影响**



![image-20250804160529904](C:\Users\10424\AppData\Roaming\Typora\typora-user-images\image-20250804160529904.png)

![image-20250804160951405](C:\Users\10424\AppData\Roaming\Typora\typora-user-images\image-20250804160951405.png)

### 镜像操作

<img src="C:\Users\10424\AppData\Roaming\Typora\typora-user-images\image-20250804172821969.png" alt="image-20250804172821969" style="zoom:67%;" />

#### 检索：

```shell
# 检索docker是否存在nginx镜像
docker search nginx
```

