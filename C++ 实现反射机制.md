# C++ 实现反射机制

## 反射：

反射(Reflection)是编程语言中一种强大的特性，它允许程序在运行时检查、修改自身的结构和行为。简单来说，反射就是程序能够"观察"和"操作"自身状态与结构的能力。



### 反射的核心功能：

1. **类型检查**：在运行时获取对象的类型信息

2. **动态调用**：在运行时调用对象的方法或访问字段

3. **结构分析**：检查类、接口、方法等结构信息

4. **动态创建**：在运行时创建新的对象实例

   

### 常见语言的反射实现：

#### Java反射示例：
```java
// 获取类信息
Class<?> clazz = Class.forName("java.util.ArrayList");

// 创建实例
List<String> list = (List<String>) clazz.newInstance();

// 调用方法
Method addMethod = clazz.getMethod("add", Object.class);
addMethod.invoke(list, "Hello");
```

#### Python反射示例：
```python
# 获取对象属性
attr = getattr(obj, 'method_name')

# 调用方法
if callable(attr):
    attr()

# 动态创建类
new_class = type('NewClass', (BaseClass,), {'x': 42})
```

### C#反射示例：
```csharp
// 获取类型信息
Type type = typeof(System.String);

// 创建实例
object instance = Activator.CreateInstance(type);

// 调用方法
MethodInfo method = type.GetMethod("ToUpper");
string result = (string)method.Invoke(instance, null);
```



### 反射的优缺点：

**优点**：
- 提高代码灵活性，支持动态行为
- 便于编写通用框架和工具(如序列化、ORM等)
- 实现插件系统和动态加载

**缺点**：
- 性能开销较大(比直接调用慢)
- 绕过编译时类型检查，可能引发运行时错误
- 代码可读性和可维护性可能降低



### 问题：

1. C++不支持反射
2. 很多业务场景需要依赖反射机制，比如：`RPC`,`WEB MVC`,对象序列化等



### 目标：

1. 类对象反射
2. 类成员数据反射
3. 类成员函数反射