## Lua 脚本的快速入门

**一.** **介绍：**

Lua是一种轻量级，高效，可以嵌入的**脚本语言**，设计用于扩展应用程序的功能。它的名字在葡萄牙语中意思为“月亮‘’，由巴西里约热内卢天主教大学（PUC-Rio）的 **TeCGraf** 小组于 **1993 年** 开发

**主要特点：**

（1）**轻量级 & 高效：**
	  非常小的代码库（整个解释器仅 几百 KB）。

  	运行速度快，通常用于 游戏、嵌入式设备、高性能应用。

（2）**可嵌入：**

​	  可以轻松集成到 C/C++/Java/Python 等程序中，作为脚本引擎使用。

​          被广泛用于游戏开发（如《魔兽世界》《愤怒的小鸟》）、服务器应用（如 Redis、Nginx）、工业软件（如 Wireshark）

（3）**动态类型：**

​	变量不需要声明类型，运行时自动确定：

```lua
local x = 10      -- 数字
x = "hello"       -- 字符串（可以随时改变类型）
```

（4）**简单易学：**
          语法类似 JavaScript/Python，但更简洁：

```lua
-- 条件判断
if age >= 18 then
    print("Adult")
else
    print("Child")
end
```

（5）**强大的表（Table）结构：**

​          Lua 的 表（Table） 可以当 数组、字典、对象 使用：

```lua
local person = {
    name = "Alice",
    age = 25,
    skills = { "Lua", "C++" }
}
print(person.name)  -- 输出 "Alice"
```

（6）**自动内存管理（垃圾回收）：**

​	  不需要手动管理内存，减少内存泄漏风险。



**二.** **网址：**https://wiki//luatos.com

​      **Lua 5.3 参考手册：**https://wiki.luatos.com/_static/lua53doc/manual.html

**三.** **基础：**

0. ***安装 Lua：***

   ```shell
   sudo apt install lua5.4
   ```

   ***运行脚本：***

   ```shell
   lua hello.lua
   ```

1. ***注释：***

   注释类型	    语法	                示例
   单行注释	    --	                -- 这是注释
   多行注释	    --[[ ... ]]	      --[[ 多行注释 ]]
   取消多行注释    ---[[ ... ]]        ---[[ 现在代码会执行 ]]

2. ***变量：***

```lua
-- 定义一个变量（默认为全局变量）
a = 1
b = 2

-- 定义一个本地的局部变量
local a = 1

-- 在Lua里没有被声明的变量都是 nil，相当于 NULL
print(a)
-- 输出: nil

-- 多重赋值
a,b = 1,2
print(a,b)
-- 输出: 1  2

-- Number数值类型
a = 1 -- a为1
a = 0x11 -- a为17
a = 2e10 -- a为2^10

-- 算术运算符
-- 支持加减乘除
-- 也支持幂运算，左移右移
print(10^5) -- 输出 100000
print(1 & 2) -- 与运算，输出 0
print(1 | 2) -- 或运算，输出 3
print(1 ~ 2) -- 异或运算，输出 3
print(10 % 2) -- 模运算，输出0
```



3. ***字符串（string）：***

```lua
-- 字符串，可以使用'' 或者 " ",也支持转义字符，如\n
a = "abcd\ndefg"
b = 'hello world!'
-- 输出: abcd
-- defg   hello world!

c = [[asddijjh\fccmioisc]]
print(c) 
-- 输出: asddijjh\fccmioisc

-- 连接符号 ..
a = 'hello'
b = 'world'
c = a .. b
print(c) -- 输出: helloworld

-- 支持字符串和数字之间的互相转化
c = tostring(10) -- 将 Number 10 转化为字符串 10
n = tonumber('10') -- 将字符串 10 转化为 Number 10;若转换失败，默认为nil
d = tonumber('abc') -- 输出 nil

-- 获取字符串长度
a = "hello"
print(#a) -- 输出5
```



4. ***function (函数)：***

   ```lua
   -- 函数
   -- 注意在lua中，函数如果没有返回值那么默认返回为nil
   -- 返回值使用return,可以返回多个值
   
   function function_name( ... )
       -- body
   end
   
   -- 写法一
   f = function(...)
        -- body
   end
   -- 写法二
   function f (...)
       -- body
   end
   
   function f(a,b)
       print(a,b)
   end
   f(1,2) -- 输出1,2
   
   function f(a,b,c)
       print(a,b,c)
   end 
   f(1,2)
   -- 输出: 1,2,nil
   
   function f(a,b)
       return a,b
   end 
   local i, j = f(1, 2)
   --i,j 分别为 1, 2
   
   ```

   

5. ***表  (table)  :***

   ```lua
   -- table的一种方式
   
   -- 表中可以存任意类型的元素
   -- 注意: 在lua里面数组下标从 1 开始
   -- 如果下标不存在的话，那么为访问该元素为 nil
   a = {1,"abc",{},function() end}
   print(a[1]) -- 输出: 1
   
   -- 可以任意赋值
   a[5] = 123 -- 那么此时a[5]为123
   
   -- 使用类似于字符串的获取长度的方法，#a
   print(#a)
   
   -- 使用Lua里面的接口来调用函数
   -- 插入元素
   a = {1,2,3,4}
   table.insert(a,5)
   print(table.concat(a, ", ")) -- print(a)只会输出a的首地址，如print(a)那么输出 table: 0x7fffbf9fdfc0
   -- 输出: 1,2,3,4,5
   
   table.insert(a,5,5) 
   print(table.concat(a, ", "))
   --指定位置， 
   -- 输出: 1,2,3,4,5,5
   
   table.insert(a,1,0) -- a = {0,1,2,3,4,5}
   print(table.concat(a, ", "))
   --指定位置， 
   -- 输出: 1,2,3,4,5,5
   
   -- 删除元素
   a = {1,2,3,4}
   table.remove(a,1) -- 这里会将第二个返回,local b = table.remove(a,1),那么b = 1
   print(table.concat(a, ", "))
   -- 输出:2, 3, 4
   ```

   ```lua
   -- table的另一种方式
   a = {
       a = 1,
       b = "123",
       c = function()
           
           end,
       d = 666 
   }
   print(a["a"]) -- 输出: 1
   print(a["b"]) -- 输出: 123
   print(a["c"]) -- 输出: function: 0x7fffc9ab3e50
   print(a.a) -- 输出: 1
   print(a.b) -- 输出: 123
   print(a.c) -- 输出: function: 0x7fffc9ab3e50
   
   a = {
       a = 1,
       b = "123",
       c = function()
           
           end,
       [",;"] = 123
   }
   print(a[",;"]) -- 输出: 123
   
   -- 可以直接对数组元素进行赋值
   a["abc"] = "abcde" -- 若没有声明某个变量，那么输出nil
   print(a["abc"]) -- 输出: abcde
   print(a.abc) -- 输出: abcde
   ```

   

6. ***全局表：_G***

   ```lua
   --_G 记录了所有声明的全局变量
   a = 1
   print(_G["a"]) -- 输出: 1
   
   -- 在Lua中table也是个全局变量，而里面的接口相当于一个函数，通过.来访问数组成员，因此可以调用table里的函数接口
   
   print(_G["table"]) -- 输出：table: 0x7fffc56c3d30
   print(_G["table"]["insert"]) -- 输出：function: 0x7fcc1baeb390
   ```

   

7. ***真和假：***

   ```lua
   -- bool类型,只有true 和 false
   -- 支持 >, <, >=, <=, ==, ~=(不等于)
   -- and &, or |,not a
   a = true;
   b = false;
   print(a and b) --false
   print(a or b) --true
   print(not b) --true
   
   -- 注意在Lua里面只有false和nil为假，其他的都是真(0也是真)
   a = 0
   print(a > 10 and "yes" or "no") -- 输出no
   ```




8. ***分支判断：***

   ```lua
   -- 写法一
   if (a > b) then ... end
       
   -- 写法二    
   if (a > b) then ...
   else ...
   end
       
   -- 写法三
   if (a > b) then ...
   elseif (a == b) then ...
   else ... 
   end
       
   -- 注意: 在Lua里面只有false和nil代表假，其他都是真，0也是真
   if 0 then print("0 is true") end --输出: 0 is true
   ```

   

9. ***循环：***

   ```lua
   -- 三种循环for,while,repeat
   
   -- 一. for循环
   for i = 1,10 do --i从1开始，到10结束，是闭区间，即[1,10],后面加一个do
       print(i)
   end
   
   for i = 1,10,2 do --i从1开始，到10结束，步长为2
       print(i)
   end
   
   for i = 10,i,-1 --i从10开始，到1结束，步长为-1
       print(i)
   end
   --注意: 在for循环里面对i进行修改是没有作用的，解释器会默认for循环里面的i是local变量
   
   -- 可以使用break来终止循环,Lua里面没有continue
   for i = 1,10 do
   	print(i)
   	if i == 5 then break end
   	-- 实现类似i == 5时continue
   	-- if i == 5 then -- 这里什么都不做
   	-- else print(i)
   	-- end 
   end
   
   -- 二. while循环
   -- 当 while 的判断满足要求时就执行，否则跳出循环
   -- while循环里面也是可以使用break来退出循环
   local n = 10
   while n > 1 do
   	-- if n == 5 then break end 
   	print(n)
   	n = n - 1  -- 注意Lua里面没有前置后置--和++
   end
   
   -- 三. repeat循环,类似于do - while式循环，先执行代码块，再检查条件，为false时退出循环
   repeat
     -- 循环体代码
   until ...
   
   -- 基本用法
   local i = 1
   repeat 
   	print(i)
   	i = i + 1
   until i > 5
   -- 输出: 1 2 3 4 5
   
   -- 模拟continue
   for i = 1, 5 do
    	repeat 
           if i == 3 then break end
           print(i)
       until true
   end
   -- 输出:1 2 4 5
   
   local input
   repeat
       print("请输入一个大于 10 的数字:")
       input = tonumber(io.read())  -- 读取用户输入
   until input and input > 10      -- 直到输入有效才退出
   print("你输入的是:", input)
   -- 输出:
   --[[
   请输入一个大于 10 的数字:
   5
   请输入一个大于 10 的数字:
   1
   请输入一个大于 10 的数字:
   10
   请输入一个大于 10 的数字:
   11
   你输入的是:     11
   ]]
   ```



10. ***字符串补充：***

    ```lua
    -- Lua里面的字符串类似于C里面的字符数组
    s = string.char(0x30,0x31,0x32,0x33)
    print(s) -- 输出: 0123
    
    n = string.byte(s,2)
    print(n) -- 输出: 49(1的ASCII码十进制值)
    
    --Lua里面ASCII为0x00的值不是结束标志，就是0
    s = string.char(0x00,0x30)
    n = string.byte(s,1)
    print(n) -- 输出: 0
    ```

    