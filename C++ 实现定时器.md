## C++ 实现定时器

1. **特点：**轻量级；高精度（毫秒）；跨平台（Linux，Windows，Mac OS等）；实现简单，使用方便
2. **设计知识：**C++11线程；chrono时间库，lambda函数，泛型编程，可变参数



#### 一. 基于多线程触发的 Timer 实现：

    1. **start：**在指定的毫秒数后调用函数
    1. **stop：**关闭定时器

```cpp
// timer.h
class Timer
{
public:
    Timer();
    Timer(int repeat);
    ~Timer();

    // 启动定时任务
    template <typename F, typename... Args>
    void start(int millisecond, F &&func, Args&&...args);

    // 关闭定时任务
    void stop();

private:
    std::thread m_thread;
    std::atomic<bool> m_active;
    std::function<void()> m_func;
    int m_period; // ms
    int m_repeat; // 触发次数，-1表示无限周期触发
};

// 启动定时任务
template <typename F, typename... Args>
void Timer::start(int millisecond, F &&f, Args&&...args) {
    if (m_active.load()) {
        return;
    }
    m_period = millisecond;
    m_func = std::bind(std::forward<F>(f), std::forward<Args>(args)...);
    m_active.store(true);
    m_thread = std::thread([&]() {
        if (m_repeat < 0) {
            while (m_active.load()) {
                std::this_thread::sleep_for(std::chrono::milliseconds(m_period));
                if (!m_active.load()) {
                    return;
                }
                m_func();
            }
        } else {
            while (m_repeat > 0) {
                if (!m_active.load()) return;
                std::this_thread::sleep_for(std::chrono::milliseconds(m_period));
                if (!m_active.load()) return;
                m_func();
                m_repeat--;
            }
        }
    });
    m_thread.detach();
}

// timer.cpp
#include <utility/timer.h>

using namespace shijie::utility;

Timer::Timer() : m_active(false), m_period(0), m_repeat(-1) {}

Timer::Timer(int repeat) : m_active(false), m_period(0), m_repeat(repeat) {

}

Timer::~Timer() {
	stop();
}

void Timer::stop() {
	m_active.store(false);
}

// main.cpp
#include <iostream>
#include <string>
#include <utility/timer.h>

using namespace std;
using namespace shijie::utility;

void foo() {
	cout << "foo" << endl;
}

void bar(const string &name) {
	cout << "bar: " << name << endl;
}

int main() {
    // 2,3为触发次数，如果不加参数那么无限执行下去
	Timer t1(2);
	t1.start(1000, foo);

	Timer t2(3);
	t2.start(1500, bar, "kitty");

	std::getchar(); // 阻塞等待输入，保持程序运行
	return 0;
}

// 输出
foo
bar: kitty
foo
bar: kitty
bar: kitty
// 一直阻塞
```



#### 二. 基于时间线触发的 Timer 实现

**TimerManager类**

（1）**schedule：**注册定时任务

（2）**update：**触发定时任务

```cpp
// timer.h
#pragma once
#include <cstdint>
#include <functional>
#include <chrono>

namespace shijie
{
	namespace timer
	{
		class Timer
		{
			friend class TimerManager;

		public:
			Timer();
			Timer(int repeat);
			~Timer();

			template <typename F, typename... Args>
			void callback(int milliseconds, F&& f, Args&&... args);
			
			void on_timer();

		private:
			static int64_t now();

		private:
			int64_t m_time; // 定时器触发的时间点，单位：毫秒
			std::function<void()> m_func;
			int m_period; // 定时器触发的周期，单位：毫秒
			int m_repeat; // 定时器触发的次数，-1表示无限触发
		};

		template <typename F, typename... Args>
		void Timer::callback(int milliseconds, F&& f, Args&&... args) {
			m_period = milliseconds;
			m_func = std::bind(std::forward<F>(f), std::forward<Args>(args)...);
		}
	}
}

// timer.h
#include <timer/timer.h>

using namespace shijie::timer;

Timer::Timer() : m_period(0), m_repeat(-1) {
	m_time = now();
}

Timer::Timer(int repeat) {
	m_time = now();
	m_repeat = repeat;
	m_period = 0;
}

Timer::~Timer() {

}

int64_t Timer::now() {
	// 获取当前时间戳
	auto now = std::chrono::system_clock::now();

	// 将当前时间戳转换成毫秒数
	auto now_ms = std::chrono::time_point_cast<std::chrono::milliseconds>(now);
	return now_ms.time_since_epoch().count();
}

void Timer::on_timer() {
	if (!m_func || m_repeat == 0) return;
	m_func();
	m_time += m_period;
	if (m_repeat > 0) {
		m_repeat--;
	}
}	

// timer_manager.h
#pragma once
#include <map>
#include <timer/timer.h>
#include <utility/singleton.h>

namespace shijie
{
	namespace timer
	{
		class TimerManager : public shijie::utility::Singleton<TimerManager>
		{
			SINGLETON(TimerManager);
		public:
			TimerManager() = default;
			~TimerManager() = default;

			template <typename F, typename... Args>
			void schedule(int milliseconds, F&& f, Args&&... args);

			template <typename F, typename... Args>
			void schedule(int milliseconds, int repeat, F&& f, Args&&... args);

			void update();

		private:
			std::multimap<int64_t, Timer> m_timers;
		};

		template <typename F, typename... Args>
		void TimerManager::schedule(int milliseconds, F&& f, Args&&... args) {
			schedule(milliseconds, -1, std::forward<F>(f), std::forward<Args>(args)...);
		}

		template <typename F, typename... Args>
		void TimerManager::schedule(int milliseconds, int repeat, F&& f, Args&&... args) {
			Timer t(repeat);
			t.callback(milliseconds, std::forward<F>(f), std::forward<Args>(args)...);
			m_timers.insert(std::make_pair(t.m_time, t));

		}
	}
}

// timer_manager.cpp
#include <timer/timer_manager.h>
using namespace shijie::timer;

void TimerManager::update() {
	if (m_timers.empty()) return;
	int64_t now = Timer::now();
	for (auto it = m_timers.begin(); it != m_timers.end(); it++) {
		if (it->first > now) {
			return;
		}
		it->second.on_timer();

		Timer t = it->second;
		it = m_timers.erase(it);
		if (t.m_repeat != 0) {
			auto new_it = m_timers.insert(std::make_pair(t.m_time, t));
			if (it == m_timers.end() || new_it -> first < it->first) {
				it = new_it;
			}
		}
	}
}
```

