### C++ 实现数据结构 vector



#### 动态数组：

1. **概念：**vector是C++容器库提供的可变大小数组的容器，采用连续空间来存储元素，可以采用下标对元素进行访问，和数组一样高效，但是它的大小是可以动态改变的。

   ---

   

2. ```cpp
   // 声明
   template<class T,class Alloc = allocator<T>>
   class vector;
   ```

   ---

   

3. **优点和缺点：**

   **优点：**元素存储在连续的空间可以随机访问；在数组最后添加和删除元素效率高

   **缺点：**在内部进行插入，删除操作效率低；当数据超过大小是要进行整体的重新分配，拷贝与释放

   ---

   

4. **容量查看：**

   ```cpp
   // 函数			含义
   // size 		 返回容器中已经使用的空间大小
   // capacity		 返回容器中已分配空间的大小
   // size <= capacity
   
   ```

   ---

   

5. **扩容因子：**vs：1.5倍；linux：2倍

   ---

   

6. **元素访问：**

   ​				  			      <img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250327174745340.png" alt="image-20250327174745340" style="zoom: 50%;" />	

   ---

   

7. **遍历：**

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250327175001727.png" alt="image-20250327175001727" style="zoom: 80%;" />

---



8. **尾部增删：**时间复杂度O(1)

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250327175221527.png" alt="image-20250327175221527" style="zoom: 80%;" />

---



  9.  **中间增删：**时间复杂度O(n)

​						        <img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250327175912214.png" alt="image-20250327175912214" style="zoom:80%;" />

****

10. **容量调整：**

​							![image-20250327180443665](C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250327180443665.png)

### 具体实现：



#### vector元素访问：

```cpp
Vector();
~Vector();

T &at(int index);
const T &at(int index) const;
T &front();
const T &front() const;
T &back();
const T &back() const;
T *data();
const T *data() const;
T &operator[](int index);
const T &operator[](int index) const;
```



### vector尾部增删：

```cpp
void push_back(const T &value) { // 尾部增加一个元素
    if (m_size < m_capacity) {
        m_data[m_size] = value;
        m_size++;
        return;
    }

    if (m_capacity == 0) { // 扩容
        m_capacity = 1;
    } else {
        m_capacity *= 2;
    }

    T *data = new T[m_capacity]; // 拷贝旧数组
    for (int i = 0; i < m_size; i++) {
        data[i] = m_data[i];
    }

    if (m_data != nullptr) { // 删除旧数组
        delete []m_data;
        m_data = nullptr;
    }

    m_data = data; // 添加新元素
    m_data[m_size] = value;
    m_size++;
}

void pop_back() { // 尾部删除一个元素
    if (m_size > 0) m_size--;
}	
```



#### 分配元素

```cpp
void assign(int n,const T &value) {
    if (m_capacity >= n) { // 当前容量可以满足分配的大小
        for (int i = 0; i < n; i++) {
            m_data[i] = value;
        }
        m_size = n;
        return;
    }

    if (m_data != nullptr) { // 删除旧数组
        delete[] m_data;
        m_data = nullptr;
        m_size = 0;
        m_capacity = 0;
    }

    while (m_capacity < n) { // 2配速扩容
        if (m_capacity == 0) {
            m_capacity = 1;
        } else {
            m_capacity *= 2;
        }
    }
    m_data = new T[m_capacity]; // 拷贝数组
    for (int i = 0; i < n; i++) {
        m_data[i] = value;
    }
    m_size = n;
}
```



#### 容量调整

```cpp
void resize(int n, const T &value) {
    if (n < m_size) {
        m_size = n;
        return;
    }

    if (n < m_capacity) {
        for (int i = m_size; i < n; i++) {
            m_data[i] = value;
        }
        m_size = n;
        return;
    }

    while (m_capacity < n) {
        if (m_capacity == 0) {
            m_capacity = 1;
        } else {
            m_capacity <<= 1;
        }
    }

    T *data = new T[m_capacity];
    for (int i = 0; i < m_size; i++) {
        data[i] = m_data[i];
    }
    for (int i = m_size; i < n; i++) {
        data[i] = value;
    }
    if (m_data != nullptr) {
        delete[] m_data;
        m_data = nullptr;
    }
    m_data = data;
    m_size = n;
}

void reserve(int n) {
    if (m_capacity > n) return;
    while (m_capacity < n) {
        if (m_capacity == 0) m_capacity = 1;
        else m_capacity <<= 1;
    }
    T *data = new T[m_capacity];
    for (int i = 0; i < m_size; i++) {
        data[i] = m_data[i];
    }
    if (m_data != nullptr) {
        delete[] m_data;
        m_data = nullptr;
    }
    m_data = data;
}

// 测试：
int main() {
	Vector<int> v;
	for (int i = 0; i < 10; i++) {
		v.push_back(i);
	}

	v.resize(12);
	v.show();

	v.reserve(15);
	v.show();

	v.reserve(30);
	v.show();
	return 0;
}
// 输出
size = 12 capacity = 16
0 1 2 3 4 5 6 7 8 9 0 0
size = 12 capacity = 16
0 1 2 3 4 5 6 7 8 9 0 0
size = 12 capacity = 32
0 1 2 3 4 5 6 7 8 9 0 0
```



#### 元素遍历

```cpp
// 迭代器
/*
正向迭代器
iterator begin() {
    return iterator(m_data);
}

iterator end() {
    return iterator(m_data + m_size);
}

常量正向迭代器
const_iterator begin() const {
    return const_iterator(m_data);
}

const_iterator end() const {
    return const_iterator(m_data + m_size);
}

const_iterator cbegin() const {
    return const_iterator(m_data);
}

const_iterator cend() const {
    return const_iterator(m_data + m_size);
}

反向迭代器
reverse_iterator rbegin() {
    return reverse_iterator(m_data + m_size - 1);
}

reverse_iterator rend() {
    return reverse_iterator(m_data - 1);
}
*/
```



#### 中间增删

```cpp
// 中间增删
iterator insert(iterator pos, const T &value) {
    return insert(pos, 1, value);
}

iterator insert(iterator pos, int n, const T &value) {
    int size = pos - begin();
    if (m_size + n <= m_capacity) {
        // 移动元素
        for (int i = m_size; i > size; i--) {
            m_data[i + n - 1] = m_data[i - 1];
        }

        for (int i = 0; i < n; i++) {
            m_data[size + i] = value;
        }
        m_size += n;
        return iterator(m_data + size);
    } 

    while (m_size + n > m_capacity) {
        if (m_capacity == 0) m_capacity = 1;
        else m_capacity <<= 1;
    }

    T *data = new T[m_capacity];
    for (int i = 0; i < size; i++) {
        data[i] = m_data[i];
    }
    for (int i = 0; i < n; i++) {
        data[size + i] = value;
    }
    for (int i = size; i < m_size; i++) {
        data[i + n] = m_data[i];
    }

    if (m_data != nullptr) {
        delete[] m_data;
        m_data = nullptr;
    }
    m_data = data;
    m_size += n;
    return iterator(m_data + size);
}

iterator erase(iterator pos) {
if (pos == end()) {
    throw std::out_of_range("out of range");
}

// 删除最后一个元素
if (end() - pos == 1) {
    m_size -= 1;
    return end();
}

int size = pos - begin();
for (int i = size; i < m_size - 1; i++) {
    m_data[i] = m_data[i + 1];
}
m_size -= 1;
return pos;
}

iterator erase(iterator first,iterator last) {
// 对应下标
int f = first - begin();
int l = last - begin();
int n = last - first;
if (is_basic_type()) {
    memmove(m_data + f, m_data + l, (m_size - l) * sizeof(T));
}
else {
    for (int i = 0; i < m_size - l; i++) {
        m_data[f + i] = m_data[l + i];
    }
}
m_size -= n;
return first;
}

void clear() {
m_size = 0; // 析构函数才将数组里面的元素都删除
}
```

