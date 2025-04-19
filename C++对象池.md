# C++ 对象池开发

一. **重载 new 和 delete**

```cpp
// A.h
class A
{
private:

public:
	A() {
		cout << "A construct" << endl;
	}
	~A() {
		cout << "A destruct" << endl;
	}

	void *operator new(size_t n) {
		cout << "A new" << endl;
		return ::malloc(n);
	}

	void operator delete(void *p) {
		cout << "A delete" << endl;
		::free(p);
	}
};
// main.cpp
#include "A.h"

int main() {
	A *a = new A();
	// a 的使用过程
	delete a;
	return 0;
}

// 输出：
A new
A construct
A destruct
A delete

```

**二.对象池设计**

###### **1. 实现方法**

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250326210815916.png" alt="image-20250326210815916" style="zoom:80%;" />

###### 2. 策略接口（模板类 Allocator）

```cpp
// 策略接口
template<typename T>
class Allocator
{
public:
    virtual T *allocate() = 0;
    virtual void deallocate(T *p) = 0;
};
// 分配内存：allocate
// 回收内存：deallocate

// 实现接口
template<typename T>
class MallocAllocator : public Allocator<T>
{
public:
    MallocAllocator() = default;
    ~MallocAllocator() = default;

    virtual T * allocate() {
        auto p = ::malloc(sizeof(T));
        return reinterpret_cast<T *>(p);
    }

    virtual void deallocate(T *p) {
        ::free(p);
    }
private:
};
```

###### 3.  **实现**

1. **普通实现**

```cpp
// 使用模板类实现 new 和 delete
class A
{
private:
	typedef ObjectPool<A, MallocAllocator<A>> ObjectPool;
	static ObjectPool pool;

public:
	A() {
		cout << "A construct" << endl;
	}
	~A() {
		cout << "A destruct" << endl;
	}

	void *operator new(size_t n) {
		cout << "A new malloc" << endl;
		return pool.allocate(n);
	}

	void operator delete(void *p) {
		cout << "A delete malloc" << endl;
		pool.deallocate(p);
	}
};
A::ObjectPool A::pool; 

// 测试
#include <test_malloc/a.h>

int main() {
	A *a = new A();
	// a 的使用过程
	delete a;
	return 0;
}

// 输出
A new malloc
A construct
A destruct
A delete malloc
```

2. **数组策略**

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250326231433854.png" alt="image-20250326231433854" style="zoom:80%;" />

```cpp
// 数组分配策略,时间复杂度 O(n)
template <typename T, int N>
class ArrayAllocator : public Allocator<T>
{
private:
    unsigned char m_data[sizeof(T) * N];
    bool m_used[N]; // 1 ：已使用

public:
    ArrayAllocator() {
        for (int i = 0; i < N; i++) { // 构造函数将m_used[i]置为 false ,初始时没有对象被使用
            m_used[i] = false;
        }
    }
    ~ArrayAllocator() = default;

    virtual T*allocate() // 分配
    {
        for (int i = 0; i < N; i++) {
            if (!m_used[i]) { // 寻找未使用的对象
                m_used[i] = true;
                return reinterpret_cast<T *>(&m_data[sizeof(T) * i]);
            }
        }
        throw std::bad_alloc();
    }


    virtual void deallocate(T *p) // 释放
    {
        auto i = ((unsigned char *)p - m_data) / sizeof(T);
        m_used[i] = false;
    }
};

// 测试
#include <test_array/a.h>

int main() {
	A *arr[max_size] = {nullptr};
	for (int i = 0; i < max_size; i++) {
		A *a = new A();
		arr[i] = a;
	}

	for (int i = 0; i < max_size; i++) {
		delete arr[i];
	}

	return 0;
}

// 输出
A new ArrayAllocator
A construct
A new ArrayAllocator
A construct
A new ArrayAllocator
A construct
A new ArrayAllocator
A construct
A destruct
A delete ArrayAllocator
A destruct
A delete ArrayAllocator
A destruct
A delete ArrayAllocator
A destruct
A delete ArrayAllocator
```



3. **堆策略**

   <img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250327103100189.png" alt="image-20250327103100189" style="zoom:80%;" />

```cpp
// 堆分配策略，时间复杂度 O(logn)
class HeapAllocator : public Allocator<T>
{
public:
    enum State
    {
        FREE = 1, // 未使用
        USED = 0, // 已使用
    };

    struct Entry
    {
        State state; // 状态
        T *p; // 对象指针

        bool operator < (const Entry &other) const { // 大根堆比较重载
            return state < other.state;
        }
    };

    HeapAllocator() {
        m_available = N;
        for (int i = 0; i < N; i++) {
            m_entry[i].state = FREE;
            m_entry[i].p = reinterpret_cast<T *>(&m_data[sizeof(T) * i]);
        }

        // 生成大根堆，保证未使用的对象放在堆顶
        std::make_heap(m_entry, m_entry + N);
    }

    ~HeapAllocator() = default;

    virtual T * allocate() {
        if (m_available <= 0) throw std::bad_alloc();

        Entry e = m_entry[0];
        std::pop_heap(m_entry, m_entry + N); // 出堆
        m_available--;
        m_entry[m_available].state = USED;
        m_entry[m_available].p = nullptr;

        return e.p;
    }

    virtual void deallocate(T *p) {
        if (p == nullptr || m_available >= N) return;

        m_entry[m_available].state = FREE;
        m_entry[m_available].p = reinterpret_cast<T *>(p);
        m_available++;
        std::push_heap(m_entry, m_entry + N); // 入堆
    }

private:
    unsigned char m_data[sizeof(T) * N];
    Entry m_entry[N];
    int m_available;
};

// 测试
#include <test_heap/a.h>

int main() {
	A *arr[max_size] = {nullptr};
	for (int i = 0; i < max_size; i++) {
		A *a = new A();
		arr[i] = a;
	}

	for (int i = 0; i < max_size; i++) {
		delete arr[i];
	}

	return 0;
}

// 输出
A new HeapAllocator
A construct
A new HeapAllocator
A construct
A new HeapAllocator
A construct
A new HeapAllocator
A construct
A destruct
A delete HeapAllocator
A destruct
A delete HeapAllocator
A destruct
A delete HeapAllocator
A destruct
A delete HeapAllocator
```



4. **栈策略**

   <img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250327110616010.png" alt="image-20250327110616010" style="zoom: 80%;" />

```cpp
// 栈策略，时间复杂度 O (1)
template<typename T,int N>
class StackAllocator : public Allocator<T>
{
    public:
        StackAllocator() {
            m_allocated = 0;
            m_available = 0;
        }

        ~StackAllocator() = default;

        virtual T* allocate() {
            if (m_available <= 0 && m_allocated >= N) {
                throw std::bad_alloc();
            }

            if (m_available > 0) {
                return m_stack[--m_available]; // 栈顶出栈
            } else {
                auto p = m_data + sizeof(T) * m_allocated;
                m_allocated++;
                return reinterpret_cast<T *>(p);
            }
        }

        virtual void deallocate(T *p) {
            m_stack[m_available++] = p; // 入栈
        }
    private:
        unsigned char m_data[sizeof(T) * N];
        std::array<T *, N> m_stack;
        int m_allocated; // 数组中可分配的未使用的对象的位置
        int m_available; // 栈中要出栈的位置
};

// 测试
#include <test_stack/a.h>

int main() {
	A *arr[max_size] = {nullptr};
	for (int i = 0; i < max_size; i++) {
		A *a = new A();
		arr[i] = a;
	}

	for (int i = 0; i < max_size; i++) {
		delete arr[i];
	}

	return 0;
}
// 输出
A new StackAllocator
A construct
A new StackAllocator
A construct
A new StackAllocator
A construct
A new StackAllocator
A construct
A destruct
A delete StackAllocator
A destruct
A delete StackAllocator
A destruct
A delete StackAllocator
A destruct
A delete StackAllocator
```



4. **区块策略**

   <img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250327153750197.png" alt="image-20250327153750197" style="zoom: 50%;" />

   

   

   <img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250327154040146.png" alt="image-20250327154040146" style="zoom:50%;" />

```cpp
// 区块策略
template<typename T,int ChunksPerBlock>
class BlockAllocator : public Allocator<T>
{
    public:
        BlockAllocator() : m_head(nullptr){}
        ~BlockAllocator() = default;

        virtual T * allocate() {
            if (m_head == nullptr) {
                if (sizeof(T) < sizeof (T *)) { // 若对象大小小于指针大小抛出异常
                    std::cerr << "object size less than pointer size" << std::endl;
                    throw std::bad_alloc();
                } 
                m_head = allocate_block(sizeof(T));
            }
            Chunk *p = m_head;
            m_head = m_head->next;
            return reinterpret_cast<T *>(p);
        }

        virtual void deallocate(T *p) {
            auto chunk = reinterpret_cast<Chunk *>(p);
            chunk->next = m_head;
            m_head = chunk;
        }

    private:
        struct Chunk
        {
            Chunk *next;
        };

        Chunk *allocate_block(size_t chunk_size) 
        {
            size_t block_size = ChunksPerBlock * chunk_size;
            auto head = reinterpret_cast<Chunk *>(::malloc(block_size));
            auto chunk = head;
            for (int i = 0; i < ChunksPerBlock; i++) { // 建立区块链
                chunk->next = reinterpret_cast<Chunk *>(reinterpret_cast<unsigned char *>(chunk) + chunk_size);
                chunk = chunk->next;
            }
            chunk->next = nullptr;
            return head;
        }

    private:
        Chunk *m_head; // 头指针
};

// 测试
#include <test_block/a.h>

const int max_size = 4;
int main() {
	A *arr[max_size] = {nullptr};
	for (int i = 0; i < max_size; i++) {
		A *a = new A();
		arr[i] = a;
	}

	for (int i = 0; i < max_size; i++) {
		delete arr[i];
	}

	return 0;
}

// 输出
A new BlockAllocator
A construct
A new BlockAllocator
A construct
A new BlockAllocator
A construct
A new BlockAllocator
A construct
A destruct
A delete BlockAllocator
A destruct
A delete BlockAllocator
A destruct
A delete BlockAllocator
A destruct
A delete BlockAllocator
```



   

4. **总结**

   1. **数组策略：**对象池大小固定，不能扩容，分配效率低，时间复杂度 O(n)
   1. **堆策略：**对象池大小固定，不能扩容，分配效率一般，时间复杂度 O(logn)
   1. **栈策略：**对象池大小固定，不能扩容，分配效率高，时间复杂度 O(1)
   1. **区块策略：**对象池可以动态扩容，分配效率高，时间复杂度 O(1)

