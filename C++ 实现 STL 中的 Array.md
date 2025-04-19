## C++ 实现 STL 中的 Array



#### 静态数组：

std::array是C++容器库提供的一个固定大小数组的容器。与其内置的数组相比，是一种更安全，更容易使用的数组类型。



#### 优点：

std::array除了有内置数组支持实际访问，效率高，存储大小固定等特点外，还支持迭代器访问，获取容量，获得原始指针等高级功能。而且它还不会退化成指针给开发人员造成困惑。



#### 缺点：

长度固定，无法扩容。



#### 元素访问：

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250404100847817.png" alt="image-20250404100847817" style="zoom:50%;" />

#### 元素遍历：

**iterator,const_iterator,reverse_iterator**

std::array提供了迭代器的方式来遍历元素：

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250404101244586.png" alt="image-20250404101244586" style="zoom:50%;" />



#### 其他函数：

std::array 支持其他一些函数：

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250404101707771.png" alt="image-20250404101707771" style="zoom:50%;" />

#### 相关代码：

```cpp
#include <cstring>
#include <stdexcept>

namespace stl
{
	template<typename T, int N>
	class Array
	{
	public:
		Array() {
			memset(m_data, 0, sizeof(T) * N);
		}

		~Array() {
			
		}

		int size() const {
			return N;
		}

		bool empty() const {
			return N == 0;
		}

		T *data() {
			return m_data;
		}

		const T *data() const {
			return m_data;
		}

		// 要求大小相同
		void swap(Array<T, N> &other) {
			if (this == &other) return;

			for (int i = 0; i < N; i++) {
				auto temp = other.m_data[i];
				other.m_data[i] = m_data[i];
				m_data[i] = temp;
			}
		}

		void fill(const T &value) {
			for (int i = 0; i < N; i++) {
				m_data[i] = value;
			}
		}

		T &front() {
			return m_data[0];
		}

		const T &front() const {
			return m_data[0];
		}

		T &back() {
			return m_data[N - 1];
		}

		const T &back() const {
			return m_data[N - 1];
		}

		T &at(int index) {
			if (index < 0 || index >= N) {
				throw std::out_of_range("out of range");
			}
			return m_data[index];
		}

		const T &at(int index) const {
			if (index < 0 || index >= N) {
				throw std::out_of_range("out of range");
			}
			return m_data[index];
		}

		T &operator[](int index) {
			// 不检查下标范围
			return m_data[index];
		}

		const T &operator[](int index) const {
			// 不检查下标范围
			return m_data[index];
		}

		void show() const {
			std::cout << "size = " << size() << std::endl;
			for (int i = 0; i < N; i++) {
				std::cout << m_data[i] << " ";
			}
			std::cout << "\n";
		}
		// todo iterator

	private:
		T m_data[N];
	};
}

#include <iostream>
#include <stl/array.h>

using namespace stl;
using namespace std;

int main() {
	Array<int, 10> a;
	a.fill(1);
	a.show();

	a[0] = 2;
	a.at(1) = 3;
	a.show();

	a.back() -= 1;
	cout << a.back();


	return 0;
}

// 输出
size = 10
1 1 1 1 1 1 1 1 1 1
size = 10
2 3 1 1 1 1 1 1 1 1
0
```

#### 

#### 迭代器：

```cpp
// 正向迭代器
#pragma once

namespace stl
{
	template<typename T>
	class ArrayIterator 
	{
		typedef ArrayIterator<T> iterator;

	public:
		ArrayIterator() : m_pointer(nullptr) {}
		ArrayIterator(T *pointer) : m_pointer(pointer) {}
		~ArrayIterator() {}

		bool operator == (const iterator& other) const {
			return m_pointer == other.m_pointer;
		}

		bool operator != (const iterator& other) const {
			return m_pointer != other.m_pointer;
		}

		iterator & operator = (const iterator & other) {
			if (this == &other) return *this;
			m_pointer = other.m_pointer;
			return *this;
		}

		// 前缀++
		iterator &operator ++ () {
			m_pointer++;
			return *this;
		}

		// 后缀++
		iterator operator ++ (int) {
			auto it = *this;
			++(*this);
			return it;
		}

		iterator operator + (int n) {
			auto it = *this;
			it.m_pointer += n;
			return it;
		}

		iterator & operator += (int n) {
			m_pointer += n;
			return *this;
		}

		// 前缀--
		iterator & operator -- () {
			m_pointer--;
			return *this;
		}

		// 后缀--
		iterator operator --(int) {
			auto it = *this;
			--(*this);
			return it;
		}

		iterator operator - (int n) {
			auto it = *this;
			it.m_pointer -= n;
			return it;
		}

		iterator operator -= (int n) {
			m_pointer -= n;
			return *this;
		}

		int operator - (const iterator &other) const {
			return m_pointer - other.m_pointer;
		}

		T &operator *() {
			return *m_pointer;
		}

		T *operator ->() {
			return m_pointer;
		}

	private:
		T *m_pointer;
	};
}

// array 类有关迭代器的函数
// 普通迭代器
iterator begin() {
    return iterator(m_data);
}

iterator end() {
    return iterator(m_data + N);
}

// 常量迭代器
const_iterator cbegin() const {
    return const_iterator(m_data);
}

const_iterator cend() const {
    return const_iterator(m_data + N);
}

// 重载const版本的begin和end，返回常量迭代器
const_iterator begin() const {
    return const_iterator(m_data);
}

const_iterator end() const {
    return const_iterator(m_data + N);
}

// 反向迭代器
reverse_iterator rbegin() {
    return reverse_iterator(m_data + N);
}

reverse_iterator rend() {
    return reverse_iterator(m_data);
}

// 测试
#include <iostream>
#include "stl/array.h"

using namespace stl;
using namespace std;

int main() {
	Array<int, 10> a;
	
	// Init array
	for (int i = 0; i < 10; i++) {
		a[i] = i + 1;
	}
	
	cout << "Array content: ";
	for (int i = 0; i < a.size(); i++) {
		cout << a[i] << " ";
	}
	cout << endl;
	
	cout << "Iterator test: ";
	for (auto it = a.begin(); it != a.end(); ++it) {
		cout << *it << " ";
	}
	cout << endl;
	
	cout << "Const iterator test: ";
	const Array<int, 10>& ca = a;
	for (auto it = ca.begin(); it != ca.end(); ++it) {
		cout << *it << " ";
	}
	cout << endl;
	
	cout << "Reverse iterator test: ";
	for (auto it = a.rbegin(); it != a.rend(); ++it) {
		cout << *it << " ";
	}
	cout << endl;
	
	return 0;
}

// 输出
Array content: 1 2 3 4 5 6 7 8 9 10 
Iterator test: 1 2 3 4 5 6 7 8 9 10
Const iterator test: 1 2 3 4 5 6 7 8 9 10
Reverse iterator test: 10 9 8 7 6 5 4 3 2 1
```

