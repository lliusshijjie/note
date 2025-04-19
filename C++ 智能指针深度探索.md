## C++ 智能指针深度探索



```cpp
// 四种智能指针
std::auto_ptr;
std::unique_ptr;
std::shared_ptr;
std::weak_ptr;
```



#### RAII （Resource Acquisition Is Initialization）：一种利用对象生命周期来控制程序资源的简单技术。

在对象构造时获取资源，接着控制对资源的访问，使之在对象的生命内始终保持有效，最后在对象析构时释放资源。

（1）不需要显示地释放资源

（2）采用这种方式，对象所需的资源在其生命期内始终保持有效



### 智能指针

所谓的智能指针本质就是一个类模板，它可以创建任意类型的指针对象，当智能指针对象使用完后，对象就会自动调用析构函数去释放该指针所指向的空间。

```cpp
// 类模板
template <typename T>
class SmartPtr
{
public:
    SmartPtr() : m_data(nullptr) {}
    SmartPtr(T *data) : m_data(data){}

    ~SmartPtr() {
        if (m_data != nullptr) {
            delete m_data;
            m_data = nullptr;
        }
    } 

    T * operator->() {
        return m_data;
    }

    T & operator * () {
        return *m_data;
    }

private : 
    T * m_data;
};
```





#### auto_ptr(C++11已经废弃)

```cpp
// auto_ptr
template <typename T>
class AutoPtr
{
public:
    AutoPtr() : m_data(nullptr) {}
    AutoPtr(T *data) : m_data(data){}
    AutoPtr(AutoPtr &other) : m_data(other.release()) {
        // 从other获取所有权，并将other的指针置为nullptr
    }

    ~AutoPtr() {
        if (m_data != nullptr) {
            delete m_data;
            m_data = nullptr;
        }
    } 

    T *get() {
        return m_data;
    }

    T *release() { // 返回一个指针，同时自己不再占用指针
        auto data = m_data;
        m_data = nullptr;
        return data;
    }

    void reset(T *data = nullptr) {
        if (m_data != data) {
            delete m_data;
            m_data = data;
        }
    }

    T * operator->() {
        return m_data;
    }

    T & operator * () {
        return *m_data;
    }

    AutoPtr & operator = (AutoPtr<T> &other) {
        if (this == &other) {
            return *this;
        }
        m_data = other.release();
        return *this;
    }

private : T * m_data;
};

// main函数
#include <iostream>
#include <utility/smart_ptr.h>
#include <utility/auto_ptr.h>

using namespace shijie::utility;
using namespace std;

class Test 
{
public:
	Test() = default;
	~Test() {
		cout << "Test is deleted" << endl;
	}

	void name(const string &name) {
		m_name = name;
	}

	string name() const {
		return m_name;
	}

private:
	string m_name;
};

int main() {
	auto p = new Test();

	AutoPtr<Test> ap(p);
	ap->name("jack");

	cout << ap->name() << endl;

	AutoPtr<Test> ap2;
	ap2 = ap;
	cout << ap2->name() << endl;
	return 0;
}
// 输出
jack
jack
Test is deleted
```



#### unique_ptr（独占式）

```cpp
// unique_ptr
template <typename T>
class UniquePtr
{
public:
    UniquePtr() : m_data(nullptr) {}
    UniquePtr(T *data) : m_data(data){}
    UniquePtr(const UniquePtr<T> &other) = delete;
    UniquePtr(UniquePtr<T> &&other) : m_data(other.release()) {}
    ~UniquePtr() {
        if (m_data != nullptr) {
            delete m_data;
            m_data = nullptr;
        }
    }

    T *get() {
        return m_data;
    }

    T *release() { // 返回一个指针，同时自己不再占用指针
        auto data = m_data;
        m_data = nullptr;
        return data;
    }

    void reset(T *data = nullptr) {
        if (m_data != data) {
            delete m_data;
            m_data = data;
        }
    }

    void swap(UniquePtr<T>& other) {
        auto data = other.m_data;
        other.m_data = m_data;
        m_data = data;
    } 

    T * operator->() {
        return m_data;
    }

    T & operator * () {
        return *m_data;
    }

    UniquePtr & operator = (const UniquePtr<T> &other) = delete {}
    UniquePtr & operator = (UniquePtr<T>&& other) {
        if (this == &other) {
            return *this;
        }
        reset(other.release());
        return *this;
    }

    T & operator[](int i) const {
        return m_data[i];
    }

    explicit operator bool() const noexcept {
        return m_data != nullptr;
    }


private:
    T * m_data;
};

// main
#include <iostream>
#include <utility/smart_ptr.h>
#include <utility/auto_ptr.h>
#include <utility/unique_ptr.h>

using namespace shijie::utility;
using namespace std;

class Test 
{
public:
	Test() = default;
	~Test() {
		cout << "Test is deleted" << endl;
	}

	void name(const string &name) {
		m_name = name;
	}

	string name() const {
		return m_name;
	}

private:
	string m_name;
};

// 此时无法在进行拷贝和赋值，通过移动的方式将所有权进行转移
int main() {
	auto p = new Test();

	UniquePtr<Test> up(p);
	up->name("jerry");
	cout << up->name() << endl;

	UniquePtr<Test> up2(up.release());
	cout << up2->name() << endl;
	return 0;
}

// 输出
jerry
jerry
Test is deleted
```



#### shared_ptr

```cpp
// shared_ptr
template<typename T> 
class SharedPtr
{
public:
    SharedPtr() : m_data(nullptr), m_count(nullptr){}
    SharedPtr(T *data) : m_data(data) {
        if (data != nullptr) {
            m_count = new int(1);
        }
    }

    SharedPtr(const SharedPtr<T>& other) : m_data(other.m_data), m_count(other.m_count) {
        if (m_data != nullptr) {
            (*m_count)++;
        }
    }

    SharedPtr(SharedPtr<T>&& other) noexcept : m_data(other.m_data), m_count(other.m_count) {
        other.m_data = nullptr;
        other.m_count = nullptr;
    } 

    ~SharedPtr() {
        if (m_data != nullptr) {
            (*m_count)--;
            if (*m_count <= 0) {
                delete m_data;
                m_data = nullptr;
                delete m_count;
                m_count = nullptr;
            }
        }
    }

    T * get() const {
        return m_data;
    }

    void reset(T *data = nullptr) {
        if (m_data == data) {
            return;
        }

        if (m_data == nullptr) {
            if (data != nullptr) {
                m_data = data;
                m_count = new int(1);
            }
            return;
        }
        (*m_count)--;
        if (*m_count <= 0) {
            delete m_data;
            m_data = nullptr;
            delete m_count;
            m_count = nullptr;
        }

        m_data = data;
        if (data != nullptr) {
            m_count = new int(1);
        }
    } 

    bool unique() const {
        if (m_data == nullptr) {
            return false;
        }
        return *m_count == 1;
    }

    int use_count() const {
        if (m_data == nullptr) {
            return 0;
        }
        return *m_count;
    }

    void swap(SharedPtr<T>& other) {
        auto data = other.m_data;
        auto count = other.m_count;
        other.m_data = m_data;
        other.m_count = m_count;
        m_data = data;
        m_count = count;
    }

    T *operator -> () const {
        return m_data;
    }  

    T & operator *() const {
        return *m_data;
    }

    explicit operator bool() const noexcept {
        return m_data != nullptr;
    }

    SharedPtr & operator = (const SharedPtr<T> &other) {
        if (this == &other) {
            return *this;
        }

        // 减少原有引用计数
        if (m_data != nullptr) {
            (*m_count)--;
            if (*m_count <= 0) {
                delete m_data;
                delete m_count;
                m_data = nullptr;
                m_count = nullptr;
            }
        }

        // 指向新对象并增加引用计数
        m_data = other.m_data;
        m_count = other.m_count;
        if (m_data != nullptr) {
            (*m_count)++;
        }

        return *this;
    }

    SharedPtr & operator = (SharedPtr<T> &&other) noexcept {
        if (this == &other) {
            return *this;
        }

        // 减少原有引用计数
        if (m_data != nullptr) {
            (*m_count)--;
            if (*m_count <= 0) {
                delete m_data;
                delete m_count;
                m_data = nullptr;
                m_count = nullptr;
            }
        }

        // 转移所有权
        m_data = other.m_data;
        m_count = other.m_count;
        other.m_data = nullptr;
        other.m_count = nullptr;

        return *this;
    }

private:
    T *m_data;
    int *m_count;
};

// main
#include <iostream>
#include <utility/smart_ptr.h>
#include <utility/auto_ptr.h>
#include <utility/unique_ptr.h>
#include <utility/shared_ptr.h>

using namespace shijie::utility;
using namespace std;

class Test 
{
public:
	Test() = default;
	~Test() {
		cout << "Test is deleted" << endl;
	}

	void name(const string &name) {
		m_name = name;
	}

	string name() const {
		return m_name;
	}

private:
	string m_name;
};

int main() {
	auto p = new Test();

	SharedPtr<Test> sp(p);
	sp->name("jerry");
	cout << sp->name() << endl;
	cout << sp.use_count() << endl;

	SharedPtr<Test> sp2;
	sp2 = sp;
	cout << sp2.use_count() << endl;
	
	return 0;
}

// 输出
jerry
1
2
Test is deleted
```



#### weak_ptr（解决shared_ptr循环引用的问题）

```cpp
#include <memory>
class B;

class A {
public:
    SharedPtr<B> b_ptr;
    ~A() { std::cout << "A被销毁" << std::endl; }
};

class B {
public:
    SharedPtr<A> a_ptr;
    ~B() { std::cout << "B被销毁" << std::endl; }
};

int main() {
    {
        SharedPtr<A> a(new A());
        SharedPtr<B> b(new B());
        
        // 创建循环引用
        a->b_ptr = b;  // a引用b
        b->a_ptr = a;  // b引用a
        
        std::cout << "a的引用计数: " << a.use_count() << std::endl;  // 输出2
        std::cout << "b的引用计数: " << b.use_count() << std::endl;  // 输出2
    }
    // 离开作用域，局部变量a和b被销毁
    // 但a和b的引用计数仍然为1，因为它们相互引用
    // 所以A和B的析构函数永远不会被调用，导致内存泄漏
    
    std::cout << "程序结束" << std::endl;
    return 0;
}
```

