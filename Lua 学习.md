## Lua 学习

#### 注释：



#### 数据类型：

八种数据类型：nil，boolean，number（双精度），string（"",'',[[]]），userdata，function，thread，table

通过type()可以查看一个数据的类型

Lua中保留字一共有22个，同时存在以'_'开头，全部都是大写字母的全局变量



#### 标识符：



#### 运算符：

算术运算符：//这个是整除，/这个是除以，5 / 2 = 2.5，5 // 2 = 2（Lua>=5.3）

.. 是字符串连接符，#是求字符串长度



#### 函数：

（1）固定参数值：不要求实参的个数必须与函数中形参的个数相同。如果实参个数小于形参个数，则系统自动使用nil填充；如果实参个数多于形参个数，多出的将被系统自动忽略

（2）可变参函数：在函数定义时不给出具体形参的个数，而是使用三个连续的点号。在函数调用时就可以向该函数传递任意个数的参数，函数可以全部接收。

（3）可以返回多个值：需要多个变量来同时接收

（4）函数也可以作为参数

```lua
function sum(a, b)
	return a + b
end

function mul(a, b)
	return a * b
end

function f(m ,n, fun)
	local ans = fun(m, n)
	print(ans)
end

f(2, 3, sum)
f(2, 3, mul)
f(2, 3, function(a,b)
	return a - b
end)
```



#### 循环：

while ... do ... end

for i = x ,x, x do end

repeat ... until ...



#### 控制语句：

break,goto,没有continue



#### 数组：

```lua
-- 二维数组
-- 声明
arr = {}

-- 赋值
for i = 1 , 3 do
	arr[i] = {}
	for j = 1, 2 do
		arr[i][j] = i * j
	end
end

-- 输出
for i = 1, #arr do
	for j = 1, #arr[i] do
		print(arr[i][j])
	end
end

-- map
t = {name = "", age = "", depart = ""}
t["gender"] = ""
t.office = ""

a = "xxx"
b = 3
c = 5
-- 类似于一个map,其key为表达式
t = {
    ["emp_"..a] = true,
    [b + c] = "hello",
    ["hi"] = 123
}
t[emp_xxx] -- true
t.emp_xxx -- true
t[8] -- hello,这里不可以t.8
t.hi -- 123
t["hi"] --123

-- 混合使用
t = {"beijing",name = "zhangsan",age = 23, "shanghai",depart = "xiaoshoubu","guangzhou","shenzhen"}
print(t[1]) -- "beijing"
print(t[2]) -- "shanghai"
print(t[3]) -- "guangzhou"
print(t[4]) -- "shenzhen"

-- 有序混合使用
t = {
    {name = "",age = },
    {name = "",age = },
    {name = "",age = },
    {name = "",age = }
}

-- 一些操作函数
table.concat()
--该函数用于将指定的数组元素进行字符串连接。连接从start索引位置到end索引位置的所有数组元素，中间使用sep隔开

table.unpack()
-- 拆包。该函数返回指定的数组中的从第i个元素到第j个元素值。i，j可以选，默认第一和最后一个元素

table.pack()
t = table.pack("a","b","c")
table.concat(t,',') -- a,b,c，这里只会将连续的元素连接
t.n -- 3,这里n为打包的元素的个数

table.maxn(t) -- 返回t中最大索引值

table.insert() --用于在指定位置pos插入值为value的一个元素。其后的元素会背后移，pos可选，默认为数组部分末尾

table.remove() --用于删除并返回表中位于pos位置的元素。其后的元素会被前移，pos可选，默认删除数组最后一个元素

table.sort() 
-- 该函数用于对指定的table的数组元素就进行升序排序，也可以按照指定函数fun(a,b)中指定的规则进行排序。fun(a,b)是一个用于比较a与b的函数，a与b代表数组中的两个相邻元素。
-- 注意：
-- 如果arr中的元素既有字符串也有数值型，那么对其排序会报错
-- 如果数组中多个元素相同，则其相同的多个元素的排序结果不确定，也就是这些元素的索引谁排前谁排后未知
-- 如果数组元素含有nil，排序报错
```



#### 迭代器：

```lua
-- 两种迭代器，pairs,ipairs,用于泛型for循环中，用于指定的table
-- pairs(table) : 仅会迭代指定table中的数组元素
-- pairs(table) : 会迭代整个table元素，无论是数组元素，还是key - value
t = {1,name = "sz",age = 21,2,depart = "xxx",3,4}
for i,v in ipairs(t) do
    print(i,v)
end
-- 输出：
1       1
2       2
3       3
4       4
for i,v in pairs(t) do
    print(i,v)
end
-- 输出：
1       1
2       2
3       3
4       4
age     21
depart  xxx
name    sz
```



#### 模块：

​	模块是Lua里面特有的数据结构，可以把一些公用的代码放在同一个文件里，以API接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度

​	模块文件主要由table组成，在table中添加相应的变量，函数，最后文件返回该table即可。如果其他文件中需要使用该模块，只需要require该模块导入即可。

```lua
-- hello.lua
rectangle = {}

rectangle.pi = 3.14

function rectangle.perimeter(a,b)
    return (a + b) * 2
end

rectangle.area = function(a, b)
    return a * b
end
return rectangle

require("hello")

print(rectangle.pi) -- 3.14
print(rectangle.perimeter(3, 5)) --16
print(rectangle.area(3, 5)) -- 15
```



#### 元表和元方法：

（1）元表中有两个重要的函数：

setmetatable(table,metatable): 将metatable指定为普通表table的原表

getmetatable(table)：获取指定普通表table的元表

（2）__index元方法：

当对table进行读取访问时，如果访问的数组索引或key不存在，那么系统就会自动调用元表的（__index）元方法。重写的方法可以是一个函数，也可以是另一张表。如果重写的（__index）元方法是函数，且有返回值，则直接返回；如果没有返回值，则返回nil.

```lua
emp = {"北京", nil, name = "张三",age = 23, "上海", depart = "销售部", "广州", "深圳"}
print(emp.x) -- nil

-- 声明一个元表
meta = {}

-- 将原始表与元表进行相关联
setmetatable(emp, meta)

-- 定义__index元方法
meta.__index = function(tab, key)
    -- 有返回值
    return "该"..key.."不存在"
    -- 无返回值
    print("该"..key.."不存在")
end

print(emp.x) -- 该 x 不存在
print(emp[2]) --该 2 不存在
```



```lua
emp = {"北京", nil, name = "张三",age = 23, "上海", depart = "销售部", "广州", "深圳"}

-- 声明一个元表
meta = {}

-- 将原始表与元表进行相关联
setmetatable(emp, meta)

other = {}
other[5] = "天津"
other[6] = "西安"

-- 指定元表为另一个普通表
meta.__index = other
print(emp[5]) --天津,当emp不存在时，会到other表里面寻找
```

（3）__newindex元方法：

当用户为table中一个不存在的索引或key**赋值**时，就会自动调用如果访问的数组索引或key不存在，那么系统就会自动调用元表的（__newindex）元方法。重写的方法可以是一个函数，也可以是另一张表。如果重写的（__newindex）元方法是函数，且有返回值，则直接返回；如果没有返回值，则返回nil。

```lua
emp = {"北京", nil, name = "张三",age = 23, "上海", depart = "销售部", "广州", "深圳"}

-- 声明一个元表
meta = {}

-- 将原始表与元表进行相关联
setmetatable(emp, meta)

-- 有返回值的情况
function meta.__newindex(tab, key, value)
    print("新增的key为"..key..",value = "..value)
    --return value
    rawset(tab, key, value)
end

emp.x = "天津" --x这个索引不存在，那么会调用元表的__newindex元方法
print(emp.x) -- nil(没有使用rawset方法)
print(emp.x) -- 天津
```



```lua
emp = {"北京", nil, name = "张三",age = 23, "上海", depart = "销售部", "广州", "深圳"}

-- 声明一个元表
meta = {}

-- 将原始表与元表进行相关联
setmetatable(emp, meta)

-- 再定义一个普通表
other = {}
-- 元表指定的另一个普通表的作用是，暂存新增加的数据
meta.__newindex = other
emp.x = "天津"

print(emp.x) -- nil
print(other.x) -- 天津
```



（4）运算符元方法:

​	如果想要为一个表扩展加号，减号，等于，小于等运算功能，则可重写相应的元方法。

​	例如，如果想要为一个table扩展加号功能，则可以重写table元表的__add元方法，而具体的运算规则，在重写的元方法中。

![image-20250411104927813](C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250411104927813.png)

```lua
emp = {"北京", nil, name = "张三",age = 23, "上海", depart = "销售部", "广州", "深圳"}

-- 声明一个元表
meta = {
    __add = function(tab, num) 
        -- 遍历tab中的所有元素
        for k ,v in pairs(tab) do
            -- 若value为数据类型，则做算数加法
            if (type(v) == "number") then
                tab[k] = v + num
            -- 若value为字符串，则做拼接操作
            elseif (type(v) == "string") then
                tab[k] = v .. num
            end
       end
       -- 返回修改过的表 
       return tab
    end
}

-- 将原始表与元表进行相关联
setmetatable(emp, meta)

empsum = emp + 5
for k,v in pairs(empsum) do
    print(k..":"..v)
end
-- 输出
1:北京5
3:上海5
4:广州5
5:深圳5
age:28
depart:销售部5
name:张三5
```



（5）__tostring元方法：

```lua
emp = {"北京", name = "张三",age = 23, "上海", depart = "销售部", "广州", "深圳"}

-- 声明一个元表
meta = {
    __add = function(tab, num) 
        -- 遍历tab中的所有元素
        for k ,v in pairs(tab) do
            -- 若value为数据类型，则做算数加法
            if (type(v) == "number") then
                tab[k] = v + num
            -- 若value为字符串，则做拼接操作
            elseif (type(v) == "string") then
                tab[k] = v .. num
            end
       end
       -- 返回修改过的表 
       return tab
    end,
    __tostring = function(tab)
        str = ""
        -- 字符串拼接
        for i, v in pairs(tab) do
            str = str.." "..i..":"..v
        end
        return str
    end
}

-- 将原始表与元表进行相关联
setmetatable(emp, meta)

empsum = emp + 5
print(emp)
print(empsum)
--1:北京5 2:上海5 3:广州5 4:深圳5 name:张三5 depart:销售部5 age:28
--1:北京5 2:上海5 3:广州5 4:深圳5 name:张三5 depart:销售部5 age:28
```



（6）__call元方法：

​	当将一个table以函数形式来使用时，系统会自动调用重写的__call元方法。该用法主要是可以简化对table的相关操作，将对table的操作与函数直接相结合

```lua
emp = {"北京", name = "张三",age = 23, "上海", depart = "销售部", "广州", "深圳"}

-- 将原始表与匿名元素相关联
setmetatable (emp, {
        __call = function(tab, num, str) 
            -- 遍历table
            for k, v in pairs(tab) do
                if (type(v) == "number") then
                    tab[k] = v + num;
                elseif (type(v) == "string") then
                    tab[k] = v .. str
                end
            end
            
            return tab
        end
    })

newemp = emp(5,"-hello")
for i, v in pairs(newemp) do
    print(i..":"..v)
end
-- 输出：
1:北京-hello
2:上海-hello
3:广州-hello
4:深圳-hello
depart:销售部-hello
age:28
name:张三-hello
```



（7）元表单独定义：

​	为了便于管理与复用，可以将元素单独定义为一个文件。该文件仅可定义一个元素，且一般文件名与元表名称相同。

​	若一个文件想要使用其他文件中定义的元素，只需要使用require"元表文件名"即可将元素导入使用。如果用户想扩展该元表而又不想修改元表元素，则可在用户自己文件中重写其相应功能的元方法即可。



#### 面向对象：

​	通过table，function与元表可以模拟和构造出类似功能的结构。

（1）简单对象的创建

```lua
-- 创建一个名称为animal的对象
animal = {name = "Tom", age = 5}

-- 为aniaml添加方法
animal.bark = function(self, voice) 
    print(self.name.."在"..voice.."叫")
end

-- 另一种写法,正确写法,冒号中会自动包含一个self参数，表示当前对象本身，相当于this
function animal:bark(voice)
    print(self.name.."在"..voice.."叫")
end

-- 下面方式不被识别,会报错
animal:bark = function(voice)
    print(self.name.."在"..voice.."叫")
end
    
animal.bark("mm")
animel2 = animal

-- 将animal置空
animal = nil

animal.bark(animal, "ww")
animal:bark("ww")
```

（2）类的创建

```lua
-- 创建一个类
Animal = {name = "no_name", age = 0}
function Animal:bark(voice)
    print(self.name.."zai"..voice.."jiao")
end

--为该类添加一个无参构造器
function Animal:new()
    --创建一个空表
	local a = {}
    
    --为新表指定元表为当前基础类表
    --在新表中找不到的key，会从self基础类表中查找
    setmetatable(a, {__index = self})
    
    --返回新表
    return a
end

animal = Animal:new()
animal2 = Animal:new()

animal.name = "Tom"
animal.age = 8
animal.type = "猫"
print(animal.name.."今年"..animal.age.."岁，它是一只"..animal.type)

function animal2:skill()
   return "小老鼠，会偷油" 
end

print(animal2.name.."今年"..animal2.age.."岁，它是一只"..animal2:skill())
```



#### 类的继承

```lua
Animal = {name = "no_name", age = 0}
function Animal:bark(voice)
    print(self.name.."zai"..voice.."jiao")
end

--为该类添加一个无参构造器
function Animal:new()
    --创建一个空表
	local a = {}
    
    --为新表指定元表为当前基础类表
    --在新表中找不到的key，会从self基础类表中查找
    setmetatable(a, {__index = self})
    
    --返回新表
    return a
end

-- 为该类添加一个带参构造器（下面举例是参数为table的情况）
function Animal:new(obj)
    local a = obj or {}
    setmetatable(a, {__index = self})
    
    --返回新表
    return a
end

animal3 = Animal:new({type = "老鼠"})
animal3.name = "Jerry"
animal.age = 5

print(animal3.name.."今年"..animal3.age.."岁，它是一只"..animal3.type)
-- Jerry今年5岁，它是一只老鼠

-- 为Animal类创建一个子类Cat
Cat = Animal:new({type = "波斯猫"})
Cat.eyes = "蓝眼睛"

tomcat = Cat:new()
tomcat.name = "Tom"

-- Tom是蓝眼睛的波斯猫
print(tomcat.name.."是"..tomcat.eyes.."的"..tomcat.type)
```



#### 协同线程与协同函数

（1）协同线程

​	  Lua中有一个特殊的线程，称为coroutiue，协同线程，简称协程。其可以在运行暂停时执行，然后转去执行其他线程，然后还可返回再继续执行没有执行完毕的内容。

​          协同线程也称为协同多线程，在Lua中表示独立的执行线程。任意时刻只会有一个协程执行，而不会出现多个协程同时执行的情况。

​	   协同线程的类型为thread，其启动，暂停，重启等，都需要通过函数来控制。

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250413112749008.png" alt="image-20250413112749008" style="zoom:67%;" />

```lua
-- 创建一个协同线程实例
crt = coroutine.create(
	function (a, b)
        print(a, b, a + b)
        -- 获取正在运行的协同线程实例，thread类型
        tr = coroutine.running()
        print(tr)
        
        -- 查看协同线程实例的类型
        print(type(tr))
        
        -- 查看协同线程实例的状态
        print(coroutine.status(crt))
        
        -- 将当前协同线程实例挂起
        coroutine.yield()
        print("又重新返回到了协同线程")
    end
)

-- 启动协同线程实例
coroutine.resume(crt, 3, 5)

-- 查看crt的类型
print("main-"..type(crt))

-- 查看协同线程实例的状态
print("main-"..coroutine.status(crt))

-- 继续执行系统线程
coroutine.resume(crt, 3, 5)

-- 查看协同线程实例的状态
print("main-"..coroutine.status(crt))

-- 输出
thread: 0x5557bee54d48
thread
running
main-thread
main-suspended
又重新返回到了协同线程
main-dead


```



```lua
-- 创建一个协同线程实例
crt = coroutine.create(
	function (a, b)
        print(a, b)
        -- 将当前协同线程实例挂起
        -- 在挂起时携带两个返回值,a * b, a / b
        coroutine.yield(a * b, a / b)
        print("又重新返回到了协同线程")
        return a + b, a - b
    end
)
-- 启动协同线程，返回三个值
-- 第一个返回值为协同线程启动成功状态
-- 后面的返回值为内置函数的返回值
success, res1, res2 = coroutine.resume(crt, 12, 3)
print(success, res1, res2)
-- 输出
-- 12      3
-- true    36      4.0

success, res1, res2 = coroutine.resume(crt, 12, 3)
print(success, res1, res2)
-- 输出
-- 又重新返回到了协同线程
-- true    15（12 + 3）      9（12 - 3）
```



（2）协同函数

​	  协同线程可以单独创建执行，也可以通过协同函数的调用启用执行。使用coroutine的

wrap()函数创建的就是协同函数，其类型为function。

​	   由于协同函数的本质就是函数，所以协同函数的调用方式就是标准的函数调用方式。只不过，协同函数的调用会启动其内置的协同线程。



```lua
-- 创建一个协同函数
cf = coroutine.wrap(
	function (a, b)
        print(a, b)
        
        -- 获取当前协同函数创建的协同线程
        tr = coroutine.running()
        print("tr的类型为："..type(tr))
        
        -- 挂起
        coroutine.yield(a + 1, b + 1)
        print("又重新返回到了协同线程")
        
        return a + b, a - b
    end
)

cftr, res1, res2 = cf(3, 5)
print(res1, res2)

print("cf的类型为："..type(cf))

-- 重启挂起的协同线程
res1, res2 = cf()
print(res1, res2)

-- 输出
-- 3       5
-- tr的类型为：thread
-- 4       6
-- cf的类型为：function
-- 又重新返回到了协同线程
-- 8       -2



-- 另一种重启方式
-- 创建一个协同函数
cf = coroutine.wrap(
	function (a, b)
        print(a, b)
        
        -- 获取当前协同函数创建的协同线程
        tr = coroutine.running()
        print("tr的类型为："..type(tr))
        
        -- 挂起
        coroutine.yield(tr, a + 1, b + 1)
        print("又重新返回到了协同线程")
        
        return a + b, a - b
    end
)

cftr, res1, res2 = cf(3, 5)
print(res1, res2)

print("cf的类型为："..type(cf))

-- 重启挂起的协同线程
success, res1, res2 = coroutine.resume(cftr)
print(res1, res2)

-- 输出
-- 3       5
-- tr的类型为：thread
-- 4       6
-- cf的类型为：function
-- 又重新返回到了协同线程
8       -2
```



#### 文件IO

Lua中提供大量对文件进行Io操作的函数。这些函数分为两类：静态函数和实例函数。所谓静态函数是指通过io.xxx()方式对文件进行操作的函数，而实例函数则是通过Lua中面向对象方式操作的函数。

```lua
-- 以只读方式打开一个文件
file = io.open("info.properties", "r")

-- 指定要读取的文件为file
io.input(file)

-- 读取一行数据
line = io.read("*l")

-- 只要有数据，就一直读下去
while line ~= nil do
    print(line)
    line = io.read("*l")
end

-- 关闭文件
io.close(file)
```

```lua
-- 以追加方式打开一个文件
file = io.open("info.properties", "a")

-- 指定要写入的文件为file
io.output(file)

-- 写入一行数据
io.write("\ngender = male")

io.close(file)
```



实例函数实例：

```lua
-- 打开文件
file = io.open("info.properties","r")
-- 只读方式
line = file:read("*l")

while line ~= nil do
    print(line)
    line = file:read("*l")
end
-- 关闭文件
file:close()

-- 追加方式
file = io.open("info.properties", "a")
file:write("\nlevel = p7")
file:close()

-- seek()
-- 只读方式打开
file = io.open("nums.txt", "r")
pos = file:seek()
print("当前位置:" ..pos)

-- 读取一行数据
line = file:read("*l")
print(line)

pos = file:seek()
print("读取过一行数据后的位置:"..pos)

--[[
pos = file:seek("set")
print("返回到文件起始位置: "..pos)
]]--
pos = file:seek("set", 2)
print("返回到文件起始位置的后面第二个位置: "..pos)

-- 读取一行数据
line = file:read("*l")
print(line)

pos = file:seek("end", -2)
print("定位到文件最后位置的前面第二个位置: "..pos)

-- 读取一行数据
line = file:read("*l")
print(line)

file:close()
--[[
当前位置: 0
12345
读取过一行数据后的位置:6
返回到文件起始位置的后面第二个位置: 2
345
定位到文件最后位置的前面第二个位置: 10
90
]]--



```

