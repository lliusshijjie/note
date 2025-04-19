## Lua 进阶



#### 1. require多文件调用

```lua
-- require运行指定文件，并且末尾不带拓展名
-- 目录层级用"."分隔
-- 只会运行一次
require("hello") -- 调用hello.lua文件
require("path.hello2") -- 调用path文件夹里面的hello2文件

-- 从package.path中的路径里查找
package.path = package.path .. ";./path/?.lua"
require("hello2") -- 直接从path文件夹里面寻找hello2文件

-- 怎么多次调用
-- hello.lua文件
local hello = {}

function hello.say()
	print("Hello, World!")
end

return hello

-- test.lua文件
local hello = require("hello")
hello.say() -- 输出：hello,World！
```



#### 2. 迭代table

```lua
t = {"a", "b", "c", "d"}
for i = 1, #t do
	print(i, t[i])
end

-- 使用迭代器,连续数组
for i,j in ipairs(t) do
    print(i,j)
end

-- 对于不连续的数组，如果遇到不存在的下标，默认为nil，然后停止循环
t = {
    [1] = "a",
    [2] = "b",
    [3] = "c",
    [5] = "d"
}
for i,j in ipairs(t) do
    print(i,j)
end 
-- 输出：
-- 1 a
-- 2 b
-- 3 c

-- 对于字符串类型的数组
t = {
    apple = "a",
    banana = "b",
    eraser = "c",
    water = "d"
}
for i,j in pairs(t) do
    print(i,j)
end
-- 输出：
-- water   d
-- apple   a
-- eraser  c
-- banana  b 

t = {a = 1, b = 2, c = 3, d = 4}
-- 使用pairs可以将所有元素打印出来,pairs内部使用next函数
-- next(t) --输出:a 1
-- next(t,"a") --输出: b 2
-- next(t, "d") --输出: nil
-- 对于一个空表，使用next(t)会直接输出nil，因此可以判断一个表是否是空表
```



#### 3. string

```lua
-- 字符串正数下标从1开始，字符串右边下标从-1往左递增
local s = "123456789"
print(string.byte(s,1))
print(s:byte(1))
-- 输出：
-- 49 49

print(s:byte(1,-1))
-- 49 50 51 52 53 54 55 56 57

print(s:byte())
-- 49

string.char(0x35,0x36,0x37) --53 54 55

local f = string.format("%d,%d",1,2)
print(f) -- 1,2

-- 字符串长度
s:len()
#s

-- 字符串中的大写字母变成小写字母,返回一个副本
local s = "AbC"
s:lower() -- 大变小
s:upper() -- 小变大

-- 大小端
local b = string.pack("<i",10) -- 小端
print(b:byte(1,-1)) -- 10 0 0 0
local b = string.pack("<ii",10,20) 
print(b:byte(1,-1)) -- 10 0 0 0 20 0 0 0

-- rep重复
print(string.rep("123",3)) -- 123123123
print(string.rep("123",3,",")) --123,123,123

--reverse反转st
print(string.reverse("abc")) --cba

-- sub返回子串
local s = "123456"
print(s:sub(4)) -- 456
print(s:sub(4,5)) --[4,5],45
```



#### 4. 正则

![image-20250408194206833](C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250408194206833.png)

![image-20250408195136474](C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250408195136474.png)

```lua
string.find()
string.match()

local s = "abcd1234"
print(string.find(s,"123")) --5 7
print(string.match(s,"123")) --123,方便正则匹配

-- %转义字符,%.表示要匹配一个.
-- [],括号中进行连续匹配
-- [^],不匹配
-- 使用()进行多次匹配
local s = "abcd1234"
string.gsub(s,"%d","x") -- 将所有数字都变成x，返回替换后的字符串和一共替换多少次，即abcdxxxx 4

string.gmatch(s,"a%d") --匹配所有的a数字的字符串
s = "a1a2a3a4a5"
for w in string.gmatch(s,"a%d") do
    print(w)
end -- a1 a2 a3 a4 a5
```



#### 5. 元表，元方表，面向对象

```lua
t = {a = 15}
-- 元方法
mt = {
    __add = function(a, b)
        return a.a + b
    end,
}
setmetatable(t, mt)
print(t + 1) -- 16

t = {a = 15}
mt = {
    __index = function(table, key)
        return 123
    end,
}
setmetatable(t, mt)
print(t["abc"]) -- 不存在，返回123

t = {a = 15}
mt = {
    __index = {
        abc = 123,
        def = 456,
    }
}
setmetatable(t, mt)
print(t["abc"]) -- 123
print(t["def"]) -- 456

t = {a = 15}
mt = {
    __newindex = function(t, k, v)
        rawset(t,k,v) -- 如果不加这个方法，那么下面的t["abc"] = 1赋值无效，输出nil
        -- 如果不使用这个rawset方法，例如使用t[k] = v，那么会无限使用__newindex函数，而爆栈
    end
}
setmetatable(t, mt)
t["abc"] = 1
print(t["abc"]) -- 1


t = {
	a = 0,
	add = function(tab, sum) 
		tab.a = tab.a + sum;
	end
}
t:add(10) -- 语法糖，t.add(t, 1)当 t 和调用方法的第一个参数一致时，等价于 t:add(1)
print(t["a"])

-- 实现面向对象,bagmt相当于父类，而bag相当于子类，使用setmetatable时如果发现函数不存在，就会到bagmt里面寻找对应的函数来执行
bagmt = {
	put = function(t, item)
		table.insert(t.items, item)
	end,
	take = function(t)
		return table.remove(t.items)
	end,
	list = function(t)
		return table.concat(t.items, ", ")
	end,
	clear = function(t)
		t.items = {}
	end
}

bag = {}

bagmt["__index"] = bagmt
function bag.new()
	local t = {
		items = {}
	}
	setmetatable(t, bagmt)
	return t
end

local b = bag.new()
b:put("apple")
b:put("apple")
b:put("apple")
b:put("apple")
b:put("apple")
b:put("apple")
print(b:list())
```



#### 6. 协程 coroutine

```lua
local co = coroutine.create(function()
	print("hello world!") 
end
)

coroutine.resume(co) -- 运行协程, hello world!

local co = coroutine.create(function()
	print("hello world!") 
	coroutine.yield(1,2,3) -- 相当于暂停这个协程
	print("continue run~~~")
end
)

coroutine.resume(co) -- hello world!
coroutine.resume(co) -- continue run

local co = coroutine.create(function()
	print("hello world!") 
	local r1,r2,r3 = coroutine.yield(1,2,3) -- 相当于暂停这个协程
	print("continue run~~~",r1, r2, r3)
end
)

print("coroutine.resume", coroutine.resume(co)) --hello world! coroutine.resume true 1 2 3
coroutine.resume(co,4,5,6)  --continue run~~~ 4       5       6
```

