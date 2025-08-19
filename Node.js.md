# Node.js

菜鸟教程：https://www.runoob.com/nodejs/nodejs-tutorial.html

官方文档：https://nodejs.cn/api/

Node.js 的核心架构围绕**异步 I/O**、**事件驱动**和**轻量高效**三大特性构建。要掌握其精髓，需重点关注以下五大核心模块和设计思想：

---

## 一、核心模块

###  1. 事件循环（Event Loop）—— Node.js 的“心脏”

- **作用**：单线程处理高并发的核心机制，协调异步任务调度。
- **关键原理**：
  - **非阻塞 I/O**：文件读写、网络请求等操作不阻塞主线程，通过回调通知结果。
  - **事件队列**：异步任务完成后，回调函数进入队列，由事件循环按顺序执行。
- **典型代码**：
  ```javascript
  setTimeout(() => console.log('异步任务'), 0); 
  console.log('同步任务');
  // 输出顺序：同步任务 -> 异步任务
  ```

> ⚠️ **理解重点**：为什么 `fs.readFile` 不卡住程序？—— I/O 操作交给系统内核，主线程继续执行其他代码。

---

###  2. 模块系统（CommonJS）—— 代码组织基石
- **核心规则**：
  - `require()`：导入模块（如 `const fs = require('fs')`）。
  - `module.exports`：导出功能（替代 ES6 的 `export`）。
- **加载机制**：
  - 模块首次加载后缓存，避免重复执行。
  - 路径解析规则（`node_modules` 逐层向上查找）。

---

###  3. 核心 API 模块 —— 开发必备工具包
| **模块**     | **核心功能**                     | **高频 API**                                |
| ------------ | -------------------------------- | ------------------------------------------- |
| **`fs`**     | 文件系统操作                     | `readFile`, `writeFile`, `createReadStream` |
| **`http`**   | 构建 Web 服务器/客户端           | `createServer()`, `request()`               |
| **`path`**   | 跨平台路径处理                   | `join()`, `resolve()`, `dirname`            |
| **`stream`** | 高效处理大文件（管道式数据处理） | `pipe()`, `on('data')`                      |
| **`events`** | 自定义事件监听与触发             | `EventEmitter`, `on()`, `emit()`            |

**示例：流处理优化性能**
```javascript
// 复制大文件（避免内存溢出）
const fs = require('fs');
fs.createReadStream('input.mp4')
  .pipe(fs.createWriteStream('output.mp4'));
```

---

### 4.包管理与生态（npm）—— 扩展能力的引擎

- **npm 核心操作**：
  - `npm install`：安装依赖（`-S` 生产依赖，`-D` 开发依赖）。
  - `package.json`：定义项目元数据、依赖和脚本命令。
- **生态力量**：
  - 超 200 万个模块（如 Express、Socket.IO、Axios）。
  - 框架支持：Express（Web 服务）、Koa（高并发中间件）、NestJS（企业级）。

---

### 5. 异步编程模型 —— 规避“回调地狱”的进化
1. **回调函数（Callback）**：基础异步模式（易嵌套过深）。
   ```javascript
   fs.readFile('file.txt', (err, data) => {
     if (err) throw err;
     console.log(data);
   });
   ```
2. **Promise**：链式调用优化嵌套。
   ```javascript
   fs.promises.readFile('file.txt')
     .then(data => console.log(data))
     .catch(err => console.error(err));
   ```
3. **Async/Await（推荐）**：以同步写法写异步代码。
   ```javascript
   async function readData() {
     try {
       const data = await fs.promises.readFile('file.txt');
       console.log(data);
     } catch (err) {
       console.error(err);
     }
   }
   ```

---

### 6. 扩展知识（高阶必备）
| **领域**     | **关键技术**                                     |
| ------------ | ------------------------------------------------ |
| **进程管理** | `child_process`（多进程）、`cluster`（负载均衡） |
| **性能优化** | 内存泄漏排查、事件循环延迟监控                   |
| **安全实践** | 输入验证、依赖漏洞扫描（`npm audit`）            |
| **部署运维** | PM2 进程守护、Docker 容器化                      |

---

### 7. 核心思想总结
1. **事件驱动**：一切 I/O 皆事件（用 `EventEmitter` 理解底层）。
2. **非阻塞优先**：避免 Sync 后缀的同步 API（如 `fs.readFileSync` 会卡住进程）。
3. **流式处理**：大文件/实时数据用 Stream 分块处理。
4. **生态杠杆**：用 npm 快速集成轮子（勿重复造轮子）。

> 🔍 **学习建议**：从 `http` + `fs` 模块写一个静态文件服务器开始，逐步加入 Express 路由和数据库（如 MongoDB），90% 的 Node.js 应用由这些核心构成。



## 二、文件系统

Node.js 的文件系统（fs）模块是与文件系统交互的核心工具，提供了文件读写、目录操作、权限管理等强大功能。下面我将从多个维度详细解析这个模块：

### 1. 模块基础与引入方式

#### 1.1 模块引入
```javascript
// 标准引入方式
const fs = require('fs');

// Promise API 引入 (Node.js v10+)
const fsPromises = require('fs').promises;
```

#### 1.2 两种操作模式

- **异步操作**：非阻塞I/O，使用回调函数
- **同步操作**：阻塞式，方法名以`Sync`结尾

### 2. 核心文件操作

#### 2.1 文件读写操作

##### 异步读写
```javascript
// 读取文件
fs.readFile('example.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log('文件内容:', data);
});

// 写入文件
fs.writeFile('output.txt', '新内容', 'utf8', (err) => {
  if (err) throw err;
  console.log('文件写入成功');
});
```

##### 同步读写
```javascript
try {
  const data = fs.readFileSync('example.txt', 'utf8');
  console.log('同步读取:', data);
  
  fs.writeFileSync('output.txt', '同步写入内容');
} catch (err) {
  console.error('操作失败:', err);
}
```

##### Promise API
```javascript
async function fileOperations() {
  try {
    const data = await fsPromises.readFile('example.txt', 'utf8');
    await fsPromises.writeFile('output.txt', `更新内容: ${new Date()}`);
    console.log('文件操作成功');
  } catch (err) {
    console.error('Promise操作失败:', err);
  }
}
```

#### 2. 2 文件追加与修改

```javascript
// 追加内容
fs.appendFile('log.txt', `${new Date()} - 日志条目\n`, (err) => {
  if (err) throw err;
});

// 修改文件权限
fs.chmod('script.sh', 0o755, (err) => {
  if (err) throw err;
});
```

### 3. 目录操作

#### 3.1 目录创建与读取
```javascript
// 创建目录
fs.mkdir('new-directory', { recursive: true }, (err) => {
  if (err) throw err;
});

// 读取目录内容
fs.readdir('./', (err, files) => {
  if (err) throw err;
  console.log('当前目录内容:', files);
});
```

#### 3.2 目录删除
```javascript
// 删除空目录
fs.rmdir('empty-directory', (err) => {
  if (err) throw err;
});

// 递归删除非空目录 (Node.js v14+)
fs.rm('non-empty-directory', { recursive: true, force: true }, (err) => {
  if (err) throw err;
});
```

### 4. 文件元数据与状态

#### 4.1 获取文件信息
```javascript
fs.stat('example.txt', (err, stats) => {
  if (err) throw err;
  
  console.log('文件信息:', {
    size: stats.size,          // 文件大小(字节)
    isFile: stats.isFile(),     // 是否是文件
    isDir: stats.isDirectory(), // 是否是目录
    createdAt: stats.birthtime, // 创建时间
    modified: stats.mtime       // 修改时间
  });
});
```

#### 4.2 检查文件存在性
```javascript
fs.access('config.json', fs.constants.F_OK | fs.constants.R_OK, (err) => {
  if (err) {
    console.log('文件不存在或不可读');
  } else {
    console.log('文件存在且可读');
  }
});
```

### 5. 流式文件处理（处理大文件）

#### 5.1 读取流
```javascript
const readStream = fs.createReadStream('largefile.mp4', { highWaterMark: 16 * 1024 });

readStream.on('data', (chunk) => {
  console.log(`接收到 ${chunk.length} 字节数据`);
});

readStream.on('end', () => {
  console.log('文件读取完成');
});

readStream.on('error', (err) => {
  console.error('读取错误:', err);
});
```

#### 5.2 写入流
```javascript
const writeStream = fs.createWriteStream('output.mp4');

writeStream.on('finish', () => {
  console.log('写入完成');
});

writeStream.on('error', (err) => {
  console.error('写入错误:', err);
});
```

#### 5.3 管道操作
```javascript
// 文件复制
fs.createReadStream('source.mp4')
  .pipe(fs.createWriteStream('copy.mp4'));

// 压缩后写入
const zlib = require('zlib');
fs.createReadStream('source.log')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('source.log.gz'));
```

### 6. 高级功能

#### 6.1 文件监视
```javascript
// 监视文件变化
const watcher = fs.watch('config.json', (eventType, filename) => {
  console.log(`事件类型: ${eventType}, 文件: ${filename}`);
  
  // 10秒后停止监视
  setTimeout(() => {
    watcher.close();
    console.log('停止监视');
  }, 10000);
});
```

#### 6.2 文件描述符操作
```javascript
fs.open('data.bin', 'r+', (err, fd) => {
  if (err) throw err;
  
  const buffer = Buffer.alloc(1024);
  
  // 从指定位置读取
  fs.read(fd, buffer, 0, buffer.length, 0, (err, bytesRead) => {
    if (err) throw err;
    
    // 写入到新位置
    fs.write(fd, buffer, 0, bytesRead, 1024, (err) => {
      if (err) throw err;
      fs.close(fd, () => console.log('操作完成'));
    });
  });
});
```

### 7. 最佳实践与注意事项

1. **异步优先**：始终优先使用异步方法，避免阻塞事件循环
2. **错误处理**：所有异步操作都必须处理错误
3. **路径安全**：使用`path`模块处理路径，避免跨平台问题
4. **流式处理**：大文件操作务必使用流
5. **权限检查**：操作前检查文件权限，避免运行时错误
6. **资源清理**：及时关闭文件描述符和监视器
7. **Promise封装**：在async/await环境中使用Promise API

```javascript
// 安全路径示例
const path = require('path');

const safePath = path.join(__dirname, 'data', 'files', 'document.txt');
fs.readFile(safePath, 'utf8', (err, data) => {
  // ...
});
```

### 8. 常用常量与标志

| 常量                    | 描述         | 常用场景   |
| ----------------------- | ------------ | ---------- |
| `fs.constants.F_OK`     | 文件是否存在 | 存在性检查 |
| `fs.constants.R_OK`     | 文件是否可读 | 权限检查   |
| `fs.constants.W_OK`     | 文件是否可写 | 权限检查   |
| `fs.constants.O_RDONLY` | 只读打开     | 打开文件   |
| `fs.constants.O_WRONLY` | 只写打开     | 打开文件   |
| `fs.constants.O_CREAT`  | 不存在则创建 | 文件创建   |

### 9. 实际应用场景

1. **配置文件管理**：读写JSON/XML配置文件
2. **日志系统**：实时追加日志条目
3. **文件上传**：流式处理大文件上传
4. **数据导入/导出**：处理CSV、Excel等数据文件
5. **静态文件服务**：创建HTTP静态文件服务器
6. **数据库备份**：定时备份数据库文件

### 10. 性能优化技巧

1. **缓冲区大小**：调整`highWaterMark`优化流性能
2. **批量操作**：使用`writev`进行批量写入
3. **零拷贝**：使用`sendfile`系统调用传输文件
4. **内存管理**：避免大文件完整加载到内存
5. **并发控制**：限制同时打开的文件描述符数量

Node.js的fs模块是服务器端开发的核心工具之一，掌握其各种特性和最佳实践，能够帮助你构建高效可靠的文件处理系统。



## 三、Path 

Node.js 的 `path` 模块是处理文件路径的核心工具，它提供了一系列跨平台的方法来操作文件路径。在 Windows 和 POSIX 系统（Linux/macOS）中，路径格式有显著差异，`path` 模块能自动处理这些差异，确保代码在不同操作系统上正常运行。

### 1. 核心功能概览

| 方法类别         | 主要方法                                              | 功能描述                 |
| ---------------- | ----------------------------------------------------- | ------------------------ |
| **路径拼接**     | `path.join()`                                         | 智能拼接路径片段         |
| **路径解析**     | `path.resolve()`                                      | 解析绝对路径             |
| **路径规范化**   | `path.normalize()`                                    | 简化路径格式             |
| **路径解析**     | `path.parse()`                                        | 分解路径为对象           |
| **路径格式化**   | `path.format()`                                       | 对象转为路径字符串       |
| **路径信息提取** | `path.basename()`, `path.dirname()`, `path.extname()` | 获取文件名/目录名/扩展名 |
| **平台处理**     | `path.sep`, `path.delimiter`                          | 获取平台分隔符           |

### 2. 核心方法详解

#### 2.1 路径拼接 - `path.join()`
智能拼接多个路径片段，自动处理分隔符

```javascript
const path = require('path');

// 跨平台安全拼接
console.log(path.join('/user', 'documents', 'files/notes.txt'));
// POSIX: /user/documents/files/notes.txt
// Windows: \user\documents\files\notes.txt

// 处理相对路径
console.log(path.join('src', '../images', './logo.png'));
// 输出: images/logo.png (所有平台)
```

#### 2.2 绝对路径解析 - `path.resolve()`
从右向左解析，直到构造出绝对路径

```javascript
// 示例目录: /home/user/project
console.log(path.resolve('src', 'app.js'));
// 输出: /home/user/project/src/app.js

console.log(path.resolve('/var', 'www', '../html'));
// 输出: /var/html (所有平台)

console.log(path.resolve()); // 获取当前工作目录绝对路径
```

#### 2.3 路径规范化 - `path.normalize()`
简化路径中的冗余部分

```javascript
console.log(path.normalize('/user//docs/../files/./notes.txt'));
// 输出: /user/files/notes.txt (POSIX)
// 输出: \user\files\notes.txt (Windows)

console.log(path.normalize('C:\\user\\docs\\..\\files\\.\\notes.txt'));
// 输出: C:\user\files\notes.txt (Windows)
```

#### 2.4 路径分解与重组
##### `path.parse()` - 路径解析为对象
```javascript
const parsed = path.parse('/home/user/project/src/app.js');
console.log(parsed);
/* 输出:
{
  root: '/',
  dir: '/home/user/project/src',
  base: 'app.js',
  ext: '.js',
  name: 'app'
} */
```

##### `path.format()` - 对象重组为路径
```javascript
const formatted = path.format({
  dir: '/var/www/site',
  name: 'index',
  ext: '.html'
});
console.log(formatted); // /var/www/site/index.html
```

#### 2.5 路径信息提取
```javascript
const filePath = '/var/www/site/index.html';

// 获取文件名 (带扩展名)
console.log(path.basename(filePath)); // index.html

// 获取文件名 (不带扩展名)
console.log(path.basename(filePath, '.html')); // index

// 获取目录名
console.log(path.dirname(filePath)); // /var/www/site

// 获取扩展名
console.log(path.extname(filePath)); // .html
```

### 3. 跨平台处理技巧

#### 3.1 平台特定属性
```javascript
// 路径分隔符 (POSIX: /, Windows: \)
console.log('分隔符:', path.sep);

// 环境变量分隔符 (POSIX: :, Windows: ;)
console.log('环境变量分隔符:', path.delimiter);

// 示例：拆分PATH环境变量
const PATH = process.env.PATH;
console.log(PATH.split(path.delimiter));
```

#### 3.2 平台兼容性最佳实践
```javascript
// 错误做法：手动拼接路径
const badPath = 'docs' + '\\' + 'files\\data.txt'; // Windows专用

// 正确做法：使用path.join()
const goodPath = path.join('docs', 'files', 'data.txt'); // 跨平台安全

// 处理Windows路径斜杠问题
const winPath = 'C:\\Users\\Project\\file.txt';
const posixPath = winPath.split(path.sep).join('/');
console.log(posixPath); // C:/Users/Project/file.txt
```

#### 3.3 相对路径转绝对路径
```javascript
function safeResolve(relativePath) {
  return path.isAbsolute(relativePath) 
    ? relativePath
    : path.resolve(process.cwd(), relativePath);
}

console.log(safeResolve('src/app.js')); // 转为绝对路径
console.log(safeResolve('/usr/local/bin')); // 保持绝对路径
```

### 4. 高级应用场景

#### 4.1 动态模块加载
```javascript
function loadModule(moduleName) {
  // 安全构建模块路径
  const modulePath = path.join(__dirname, 'modules', `${moduleName}.js`);
  
  // 检查路径是否在允许目录内
  if (!modulePath.startsWith(path.join(__dirname, 'modules'))) {
    throw new Error('非法模块路径!');
  }
  
  return require(modulePath);
}
```

#### 4.2 文件扩展名处理
```javascript
function changeExtension(filePath, newExtension) {
  const { dir, name } = path.parse(filePath);
  return path.format({
    dir,
    name,
    ext: newExtension.startsWith('.') ? newExtension : `.${newExtension}`
  });
}

console.log(changeExtension('/docs/report.txt', '.pdf')); 
// /docs/report.pdf
```

#### 4.3 路径安全验证
```javascript
function isSafePath(baseDir, targetPath) {
  // 解析为绝对路径
  const absBase = path.resolve(baseDir);
  const absTarget = path.resolve(targetPath);
  
  // 检查目标路径是否在基础目录内
  return absTarget.startsWith(absBase + path.sep);
}

// 示例验证
console.log(isSafePath('/var/www', '/var/www/uploads/image.jpg')); // true
console.log(isSafePath('/var/www', '/etc/passwd')); // false
```

### 5. 特殊路径处理

#### 5.1 当前目录与父目录
```javascript
// 包含相对路径的解析
console.log(path.resolve('/foo', './bar')); // /foo/bar
console.log(path.resolve('/foo', '../bar')); // /bar
console.log(path.normalize('/foo/bar/../baz')); // /foo/baz
```

#### 5.2 空字节防护
```javascript
// 安全处理用户输入路径
function safePath(input) {
  // 过滤空字节（常见攻击手段）
  if (input.includes('\0')) {
    throw new Error('非法路径: 包含空字节');
  }
  return path.normalize(input);
}
```

### 6. 常见问题解决方案

#### 问题1：路径分隔符不一致
**解决方案**：始终使用 `path.join()` 替代手动拼接

#### 问题2：相对路径定位错误
**解决方案**：使用 `path.resolve()` 结合 `__dirname`
```javascript
// 获取当前模块所在目录的绝对路径
const configPath = path.resolve(__dirname, 'config.json');
```

#### 问题3：Windows路径在POSIX系统出错
**解决方案**：使用 `path.win32` 或 `path.posix` 子模块
```javascript
// 强制使用Windows格式
const winStylePath = path.win32.join('C:', 'Program Files', 'app.exe');

// 强制使用POSIX格式
const posixStylePath = path.posix.join('/usr', 'local', 'bin');
```

### 7. 最佳实践总结

1. **禁止手动拼接路径**：始终使用 `path.join()` 或 `path.resolve()`
2. **使用 `__dirname` 定位当前文件**：避免依赖 `process.cwd()`
3. **用户输入路径验证**：检查空字节和路径穿越（`..`）
4. **显式处理扩展名**：使用 `path.extname()` 替代字符串操作
5. **跨平台考虑**：
   - 测试 Windows 和 POSIX 路径
   - 使用 `path.sep` 替代硬编码分隔符
6. **路径标准化**：操作前使用 `path.normalize()`
7. **路径安全**：验证路径是否在允许目录内

通过合理运用 `path` 模块，您可以编写出健壮、跨平台的 Node.js 应用程序，有效避免路径相关的常见错误和安全问题。



## 四、HTTP 

Node.js 的 `http` 模块是构建网络应用的核心模块，它允许创建 HTTP 服务器和客户端。这个模块提供了处理 HTTP 请求和响应的底层功能，是构建 Web 服务器、API 服务、代理服务器等的基础。

### 1. HTTP 模块基础

#### 1.1  模块引入
```javascript
const http = require('http');
```

### 1.2 核心功能
- 创建 HTTP 服务器
- 发起 HTTP 客户端请求
- 处理 HTTP 请求和响应
- 管理 HTTP 头信息

### 2. 创建 HTTP 服务器

#### 2.1 基本服务器创建
```javascript
const server = http.createServer((req, res) => {
  // 请求处理逻辑
  res.end('Hello, World!');
});

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

#### 2.2 服务器方法
| 方法                  | 描述                 |
| --------------------- | -------------------- |
| `server.listen()`     | 启动服务器监听连接   |
| `server.close()`      | 停止服务器接受新连接 |
| `server.setTimeout()` | 设置套接字超时时间   |

### 3. 请求对象 (IncomingMessage)

当接收到请求时，回调函数的第一个参数是 `req` (http.IncomingMessage) 对象：

#### 3.1 关键属性
| 属性              | 描述            | 示例                                     |
| ----------------- | --------------- | ---------------------------------------- |
| `req.method`      | HTTP 请求方法   | 'GET', 'POST'                            |
| `req.url`         | 请求的 URL 路径 | '/api/users'                             |
| `req.httpVersion` | HTTP 协议版本   | '1.1'                                    |
| `req.headers`     | HTTP 请求头对象 | `{ 'content-type': 'application/json' }` |

#### 3.2 读取请求体
```javascript
let body = [];
req.on('data', (chunk) => {
  body.push(chunk);
}).on('end', () => {
  body = Buffer.concat(body).toString();
  console.log('Request body:', body);
});
```

### 4. 响应对象 (ServerResponse)

回调函数的第二个参数是 `res` (http.ServerResponse) 对象：

#### 4.1 核心方法
| 方法                                   | 描述               |
| -------------------------------------- | ------------------ |
| `res.writeHead(statusCode[, headers])` | 发送响应头         |
| `res.write(data)`                      | 发送响应内容       |
| `res.end([data])`                      | 结束响应           |
| `res.setHeader(name, value)`           | 设置响应头         |
| `res.getHeader(name)`                  | 获取已设置的响应头 |

#### 4.2 发送响应示例
```javascript
// 简单响应
res.end('Hello World');

// 设置状态码和头部
res.writeHead(200, {
  'Content-Type': 'text/plain',
  'X-Custom-Header': 'value'
});
res.end('Response with headers');

// 分块发送响应
res.write('First part');
res.write('Second part');
res.end('Final part');
```

### 5. HTTP 客户端功能

#### 5.1 发起 GET 请求
```javascript
http.get('http://example.com', (res) => {
  let data = '';
  
  res.on('data', (chunk) => {
    data += chunk;
  });
  
  res.on('end', () => {
    console.log('Response:', data);
  });
}).on('error', (err) => {
  console.error('Error:', err.message);
});
```

#### 5.2 发起 POST 请求
```javascript
const postData = JSON.stringify({ title: 'Node.js Guide' });

const options = {
  hostname: 'api.example.com',
  port: 80,
  path: '/data',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Content-Length': Buffer.byteLength(postData)
  }
};

const req = http.request(options, (res) => {
  console.log(`Status Code: ${res.statusCode}`);
  
  res.setEncoding('utf8');
  res.on('data', (chunk) => {
    console.log(`Response: ${chunk}`);
  });
});

req.on('error', (err) => {
  console.error(`Request error: ${err.message}`);
});

req.write(postData);
req.end();
```

### 6. 高级特性与技巧

#### 6.1 处理文件上传
```javascript
const server = http.createServer((req, res) => {
  if (req.url === '/upload' && req.method === 'POST') {
    const fileStream = fs.createWriteStream('upload.txt');
    req.pipe(fileStream);
    
    req.on('end', () => {
      res.writeHead(201, { 'Content-Type': 'text/plain' });
      res.end('File uploaded');
    });
  }
});
```

#### 6.2 创建代理服务器
```javascript
const proxy = http.createServer((clientReq, clientRes) => {
  const options = {
    hostname: 'target-server.com',
    port: 80,
    path: clientReq.url,
    method: clientReq.method,
    headers: clientReq.headers
  };

  const proxyReq = http.request(options, (proxyRes) => {
    clientRes.writeHead(proxyRes.statusCode, proxyRes.headers);
    proxyRes.pipe(clientRes);
  });

  clientReq.pipe(proxyReq);
});

proxy.listen(8080);
```

#### 6.3 处理 HTTPS
```javascript
const https = require('https');

// 创建HTTPS服务器
const options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem')
};

https.createServer(options, (req, res) => {
  res.writeHead(200);
  res.end('Secure connection!');
}).listen(443);
```

#### 6.4 处理超时
```javascript
server.setTimeout(5000, (socket) => {
  console.log('Request timed out');
  socket.end();
});
```

### 7. 性能优化技巧

1. **使用连接重用**：
   ```javascript
   // 客户端设置keep-alive
   const agent = new http.Agent({ keepAlive: true });
   http.request({ agent }, ...);
   ```

2. **响应流处理**：
   ```javascript
   // 使用流处理大文件
   const fileStream = fs.createReadStream('largefile.txt');
   fileStream.pipe(res);
   ```

3. **请求体延迟处理**：
   
   ```javascript
   // 仅当需要时处理请求体
   if (req.method === 'POST' && req.url === '/upload') {
     // 处理请求体
   }
   ```

### 8. 安全最佳实践

1. **设置安全头部**：
   
   ```javascript
   res.setHeader('Content-Security-Policy', "default-src 'self'");
   res.setHeader('X-Content-Type-Options', 'nosniff');
   res.setHeader('Strict-Transport-Security', 'max-age=31536000');
   ```
   
2. **防止 HTTP 参数污染**：
   ```javascript
   // 处理重复的查询参数
   const { searchParams } = new URL(req.url, `http://${req.headers.host}`);
   const param = searchParams.get('param');
   ```

3. **验证输入数据**：
   ```javascript
   // 限制请求体大小
   const MAX_BODY_SIZE = 1024 * 1024; // 1MB
   if (req.headers['content-length'] > MAX_BODY_SIZE) {
     res.writeHead(413).end();
     return;
   }
   ```

### 9. 实际应用场景

#### 9.1 RESTful API 服务器
```javascript
const server = http.createServer((req, res) => {
  const { method, url } = req;
  
  if (method === 'GET' && url === '/api/users') {
    // 获取用户列表
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify([{ id: 1, name: 'Alice' }]));
  } 
  
  else if (method === 'POST' && url === '/api/users') {
    // 创建新用户
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      const newUser = JSON.parse(body);
      // 保存用户...
      res.writeHead(201).end();
    });
  }
  
  else {
    res.writeHead(404).end();
  }
});
```

#### 9.2 静态文件服务器
```javascript
const server = http.createServer((req, res) => {
  const filePath = path.join(__dirname, 'public', req.url);
  
  fs.stat(filePath, (err, stats) => {
    if (err) {
      res.writeHead(404).end('File not found');
      return;
    }
    
    if (stats.isDirectory()) {
      res.writeHead(403).end('Directory access forbidden');
      return;
    }
    
    const readStream = fs.createReadStream(filePath);
    readStream.pipe(res);
  });
});
```

### 10. 与 Express 框架的关系

虽然 Express 等框架提供了更高级的抽象，但了解 http 模块至关重要：
- Express 底层基于 http 模块构建
- http 模块提供更细粒度的控制
- 理解 http 模块有助于调试框架问题

```javascript
// Express 如何封装 http 模块
const express = require('express');
const app = express();

// 等同于
const http = require('http');
const app = http.createServer((req, res) => { ... });
```

### 11. 总结

Node.js 的 http 模块提供了强大的网络功能：
1. **核心能力**：创建服务器和客户端，处理 HTTP 协议
2. **关键对象**：IncomingMessage（请求）和 ServerResponse（响应）
3. **进阶特性**：流处理、HTTPS、代理服务器、文件上传
4. **性能优化**：连接重用、流式响应
5. **安全实践**：安全头部、输入验证

掌握 http 模块是构建高效、安全网络应用的基础，即使在使用高级框架时，这些底层知识也能帮助你更好地理解和优化应用行为。



## 四补、HTTP 协议格式深度解析

HTTP（超文本传输协议）是现代互联网的基础协议，理解其格式对于网络开发至关重要。HTTP 报文由**请求报文**（客户端发往服务器）和**响应报文**（服务器返回客户端）组成，两者都遵循相同的结构格式。

### 1. HTTP 报文核心结构

```http
┌───────────────────┐
│    Start Line     │  <-- 请求行/状态行
├───────────────────┤
│     Headers       │  <-- 头部字段（键值对）
├───────────────────┤
│                   │
│     Message Body  │  <-- 报文主体（可选）
│                   │
└───────────────────┘
```

### 2. HTTP 请求报文详解

#### 2.1 请求行（Request Line）
```http
GET /api/users?page=2 HTTP/1.1
```
- **请求方法**：定义操作类型
  - `GET`：获取资源
  - `POST`：提交数据
  - `PUT`：更新资源
  - `DELETE`：删除资源
  - `HEAD`：获取头信息
  - `PATCH`：部分更新
- **请求目标**：资源路径 + 查询参数
- **协议版本**：`HTTP/1.1` 或 `HTTP/2`

#### 2.2 请求头（Request Headers）
```http
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json, text/plain
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
Cache-Control: no-cache
Connection: keep-alive
Content-Length: 126
```
**常用请求头**：
| 头部字段         | 说明             |
| ---------------- | ---------------- |
| `Host`           | 目标主机（必需） |
| `User-Agent`     | 客户端信息       |
| `Accept`         | 可接受的响应类型 |
| `Content-Type`   | 请求体类型       |
| `Authorization`  | 认证信息         |
| `Cookie`         | 客户端 Cookie    |
| `Cache-Control`  | 缓存策略         |
| `Content-Length` | 请求体大小       |

#### 2.3 请求体（Request Body）
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "s3cr3tP@ss"
}
```
- **格式**：由 `Content-Type` 指定
  - `application/json`：JSON 数据
  - `application/x-www-form-urlencoded`：表单数据
  - `multipart/form-data`：文件上传
  - `text/xml`：XML 数据

#### 2.4 完整请求报文示例
```http
// 请求行
POST /api/login HTTP/1.1

// 请求头
Host: example.com
User-Agent: Mozilla/5.0
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNz
Content-Length: 56

// 请求体
{"username":"john","password":"secret"}
```

### 3. HTTP 响应报文详解

#### 3.1 状态行（Status Line）
```
HTTP/1.1 200 OK
```
- **协议版本**：`HTTP/1.1` 或 `HTTP/2`
- **状态码**：三位数字代码
  - `1xx`：信息响应
  - `2xx`：成功响应
  - `3xx`：重定向
  - `4xx`：客户端错误
  - `5xx`：服务器错误
- **状态文本**：人类可读描述

**常见状态码**：
| 状态码 | 含义                  | 典型场景       |
| ------ | --------------------- | -------------- |
| 200    | OK                    | 请求成功       |
| 201    | Created               | 资源创建成功   |
| 301    | Moved Permanently     | 永久重定向     |
| 400    | Bad Request           | 请求语法错误   |
| 401    | Unauthorized          | 未认证         |
| 403    | Forbidden             | 无访问权限     |
| 404    | Not Found             | 资源不存在     |
| 500    | Internal Server Error | 服务器内部错误 |
| 503    | Service Unavailable   | 服务不可用     |

#### 3.2 响应头（Response Headers）
```http
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Sat, 15 Jul 2023 12:00:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 132
Cache-Control: max-age=3600
Set-Cookie: sessionid=38afes7a8; Path=/; HttpOnly
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000
```
**常用响应头**：
| 头部字段                      | 说明              |
| ----------------------------- | ----------------- |
| `Server`                      | 服务器软件信息    |
| `Date`                        | 响应生成时间      |
| `Content-Type`                | 响应体类型        |
| `Content-Length`              | 响应体大小        |
| `Cache-Control`               | 缓存策略          |
| `Set-Cookie`                  | 设置客户端 Cookie |
| `Location`                    | 重定向目标位置    |
| `ETag`                        | 资源版本标识符    |
| `Access-Control-Allow-Origin` | CORS 跨域控制     |

#### 3.3 响应体（Response Body）
```json
{
  "status": "success",
  "data": {
    "id": 123,
    "username": "john_doe",
    "email": "john@example.com"
  }
}
```
- **格式**：由 `Content-Type` 决定
  - `text/html`：HTML 文档
  - `application/json`：JSON 数据
  - `image/png`：二进制图片
  - `application/pdf`：PDF 文档

#### 3.4 完整响应报文示例
```http
HTTP/1.1 201 Created
Server: Apache/2.4.41
Date: Sat, 15 Jul 2023 12:05:00 GMT
Content-Type: application/json
Location: /api/users/123
Content-Length: 89

{
  "id": 123,
  "name": "John Doe",
  "createdAt": "2023-07-15T12:05:00Z"
}
```

### 4. HTTP 报文高级特性

#### 4.1 分块传输编码（Chunked Transfer Encoding）
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

7\r\n
Hello, \r\n
6\r\n
world!\r\n
0\r\n
\r\n
```
- **适用场景**：动态生成内容（无需预知内容长度）
- **格式**：
  - 每个分块包含长度头和数据
  - 以 `0\r\n\r\n` 结束

#### 4.2 多部分报文（Multipart Messages）
```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryABC123

------WebKitFormBoundaryABC123
Content-Disposition: form-data; name="username"

John Doe
------WebKitFormBoundaryABC123
Content-Disposition: form-data; name="avatar"; filename="profile.jpg"
Content-Type: image/jpeg

<JPEG 二进制数据>
------WebKitFormBoundaryABC123--
```
- **适用场景**：文件上传、混合内容
- **格式特点**：
  - `boundary` 定义分隔符
  - 每个部分有自己的头部和内容

#### 4.3 内容协商机制
```http
GET /resource HTTP/1.1
Accept: text/html, application/xhtml+xml
Accept-Language: en-US, zh-CN;q=0.9
Accept-Encoding: gzip, deflate
```
**服务器根据以下头部返回合适资源**：
- `Accept`：内容类型
- `Accept-Language`：语言
- `Accept-Encoding`：压缩格式
- `Accept-Charset`：字符集

### 5. HTTP/2 协议增强

#### 5.1 二进制分帧层

```
┌──────────────────────────┐
│       HTTP/2 Frame      │
├──────────┬───────┬──────┤
│ Length   │ Type  │ Flags│
├──────────┼───────┼──────┤
│ R        │ Stream Identifier │
├──────────┴───────┴──────┤
│         Frame Payload   │
└──────────────────────────┘
```
- **帧类型**：
  - HEADERS：头部帧
  - DATA：数据帧
  - SETTINGS：连接设置
  - PING：心跳检测
  - GOAWAY：连接终止

#### 5.2 多路复用（Multiplexing）
```
客户端请求:
  Stream 1: GET /index.html
  Stream 3: GET /style.css
  Stream 5: GET /app.js

服务器响应:
  Stream 1: HTML内容 (200 OK)
  Stream 5: JS内容 (200 OK)
  Stream 3: CSS内容 (200 OK)
```
- 单一 TCP 连接并行处理多个请求/响应
- 解决 HTTP/1.x 队头阻塞问题

#### 5.3 头部压缩（HPACK）
- 使用静态霍夫曼编码压缩头部
- 维护头部字段表避免重复传输

#### 5.4 服务器推送（Server Push）
```http
客户端请求:
  GET /index.html

服务器响应:
  index.html + 推送:
    /style.css
    /app.js
    /logo.png
```
- 服务器主动推送关联资源
- 减少客户端请求次数

### 6. 安全增强机制

#### 6.1 HTTPS 加密传输

```
HTTP over TLS 协议栈:
┌──────────────────┐
│   HTTP 应用层    │
├──────────────────┤
│   TLS 安全层     │
├──────────────────┤
│   TCP 传输层     │
├──────────────────┤
│   IP 网络层      │
└──────────────────┘
```
- 使用 SSL/TLS 加密通信
- 防止窃听和中间人攻击

#### 6.2 安全头部配置
```http
# 内容安全策略
Content-Security-Policy: default-src 'self'

# 防止点击劫持
X-Frame-Options: DENY

# 强制HTTPS
Strict-Transport-Security: max-age=31536000

# 阻止MIME类型嗅探
X-Content-Type-Options: nosniff

# XSS防护
X-XSS-Protection: 1; mode=block
```

### 7. HTTP 报文调试工具

#### 7.1 命令行工具
```bash
# 查看原始HTTP请求
curl -v https://example.com

# 发送自定义请求
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://api.example.com
```

#### 7.2 浏览器开发者工具
- **Network 面板**：查看所有HTTP请求/响应
- **Raw Headers**：查看原始报文头
- **Preview/Response**：查看响应内容

#### 7.3 专业抓包工具
- Wireshark：底层网络分析
- Fiddler：HTTP调试代理
- Charles Proxy：跨平台抓包

### 8. HTTP 协议发展趋势

#### 8.1 HTTP/3 主要改进
- **传输层协议**：QUIC (基于UDP) 替代 TCP
- **连接迁移**：IP变化时保持连接
- **0-RTT 握手**：减少连接建立延迟
- **改进的多路复用**：解决TCP队头阻塞

#### 8.2 Web 安全增强
- **SameSite Cookies**：防御CSRF攻击
- **Subresource Integrity**：资源完整性校验
- **Cross-Origin Resource Sharing (CORS)**：安全跨域

#### 8.3 性能优化方向
- **Early Hints**：服务器提前提示资源
- **Server Timing API**：性能指标测量
- **Resource Hints**：预加载关键资源
  ```html
  <link rel="preconnect" href="https://cdn.example.com">
  <link rel="preload" href="main.js" as="script">
  ```

### 9. 总结：HTTP 协议要点

1. **报文结构**：
   - 请求：请求行 + 头 + 体
   - 响应：状态行 + 头 + 体

2. **关键组件**：
   - 方法：定义操作类型（GET/POST）
   - 状态码：响应结果（200/404）
   - 头部：元数据控制
   - 主体：实际数据负载

3. **协议演进**：
   - HTTP/1.1：持久连接、分块传输
   - HTTP/2：二进制分帧、多路复用
   - HTTP/3：基于QUIC、零RTT

4. **安全实践**：
   - 强制HTTPS
   - 安全头部配置
   - 输入验证/输出过滤

理解HTTP报文格式是Web开发的基石，无论是前端优化、API设计还是安全防护，都离不开对HTTP协议的深入掌握。随着Web技术的演进，HTTP协议仍在不断发展，持续关注其新特性将帮助开发者构建更高效、更安全的网络应用。



## 五、静态资源与动态资源

在 Node.js Web 开发中，**静态资源**和**动态资源**是两种核心资源类型，理解它们的区别和处理方式对于构建高效应用至关重要。

### 1. 静态资源详解

#### 1.1 概念与特点
- **定义**：内容固定不变的文件，每次请求返回相同内容
- **特性**：
  - 内容预先存在服务器文件系统中
  - 不依赖用户请求参数
  - 适合缓存优化
- **典型示例**：
  - 前端文件：HTML, CSS, JavaScript
  - 媒体文件：图片（JPEG, PNG）, 视频（MP4）
  - 文档：PDF, TXT
  - 字体文件：WOFF, TTF

#### 1.2 原生 Node.js 处理
```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');

const server = http.createServer((req, res) => {
  // 安全路径处理
  const safePath = path.join(__dirname, 'public', path.normalize(req.url));
  
  // 防止目录遍历攻击
  if (!safePath.startsWith(path.join(__dirname, 'public'))) {
    res.writeHead(403).end('Forbidden');
    return;
  }

  fs.stat(safePath, (err, stats) => {
    if (err || !stats.isFile()) {
      res.writeHead(404).end('Not Found');
      return;
    }

    // 设置MIME类型
    const ext = path.extname(safePath);
    const mimeTypes = {
      '.html': 'text/html',
      '.js': 'text/javascript',
      '.css': 'text/css',
      '.png': 'image/png',
      '.jpg': 'image/jpeg'
    };
    
    res.setHeader('Content-Type', mimeTypes[ext] || 'application/octet-stream');
    
    // 流式传输文件
    const stream = fs.createReadStream(safePath);
    stream.pipe(res);
  });
});

server.listen(3000);
```

#### 1.3 Express 框架处理

```javascript
const express = require('express');
const app = express();

// 基础静态文件服务
app.use(express.static('public'));

// 带缓存控制的静态服务
app.use('/static', express.static('public', {
  maxAge: '1d', // 浏览器缓存1天
  setHeaders: (res, path) => {
    if (path.endsWith('.js')) {
      res.set('Content-Encoding', 'gzip');
    }
  }
}));
```

#### 1.4 性能优化技巧
- **缓存策略**：
  - 浏览器缓存：`Cache-Control: max-age=31536000` (1年)
  - 服务器缓存：内存缓存、Redis
- **压缩传输**：
  - Gzip/Brotli 压缩静态资源
  - 图片优化：WebP 格式，图片压缩
- **CDN 分发**：
  - 使用 Cloudflare、AWS CloudFront
  - 边缘节点缓存

#### 1.5 安全考虑
- 路径安全：防止 `../../../` 目录遍历攻击
- MIME 类型嗅探防护：`X-Content-Type-Options: nosniff`
- 内容安全策略：`Content-Security-Policy` 头
- 禁用敏感文件：`.env`, `.git` 等

### 2. 动态资源详解

#### 2.1 概念与特点
- **定义**：根据请求实时生成的内容
- **特性**：
  - 内容依赖请求参数、用户状态等
  - 每次请求可能返回不同结果
  - 需要服务器端处理
- **典型示例**：
  - 用户个性化页面
  - 数据库查询结果
  - API 接口响应
  - 实时聊天消息

#### 2.2 核心处理技术

##### 路由处理
```javascript
// Express 路由示例
app.get('/user/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ error: 'User not found' });
    
    res.render('user-profile', { 
      user,
      currentDate: new Date().toLocaleDateString()
    });
  } catch (err) {
    res.status(500).json({ error: 'Server error' });
  }
});
```

##### 模板引擎渲染
```javascript
// 使用 EJS 模板引擎
app.set('view engine', 'ejs');

app.get('/products', async (req, res) => {
  const products = await Product.find({ category: req.query.category });
  res.render('products', { 
    products,
    user: req.session.user 
  });
});
```

##### API 接口
```javascript
// RESTful API 示例
app.post('/api/orders', authMiddleware, async (req, res) => {
  const newOrder = new Order({
    userId: req.user.id,
    items: req.body.items
  });
  
  const savedOrder = await newOrder.save();
  res.status(201).json(savedOrder);
});
```

### 3. 性能优化技巧

#### 3.1 缓存策略
```javascript
// Redis 缓存示例
const redis = require('redis');
const client = redis.createClient();

app.get('/popular-products', async (req, res) => {
  const cacheKey = 'popular_products';
  
  try {
    const cachedData = await client.get(cacheKey);
    if (cachedData) {
      return res.json(JSON.parse(cachedData));
    }
    
    const products = await Product.find().sort({ sales: -1 }).limit(10);
    // 缓存10分钟
    await client.setEx(cacheKey, 600, JSON.stringify(products));
    res.json(products);
  } catch (err) {
    res.status(500).json({ error: 'Database error' });
  }
});
```

#### 3.2 数据库优化
- 索引优化
- 查询分页
- 批量操作减少请求次数

#### 3.3 异步处理
```javascript
// 使用队列处理耗时任务
const Queue = require('bull');

const orderQueue = new Queue('order processing');

app.post('/api/orders', (req, res) => {
  // 快速响应客户端
  res.status(202).json({ message: 'Order received' });
  
  // 后台处理
  orderQueue.add(req.body, { 
    attempts: 3,
    backoff: 5000 
  });
});
```

### 4. 安全考虑
- **输入验证**：防止 SQL 注入、NoSQL 注入
  ```javascript
  // 使用 Joi 验证
  const Joi = require('joi');
  const schema = Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().min(8).required()
  });
  ```
- **身份认证**：JWT、OAuth
- **速率限制**：防止暴力破解
  ```javascript
  const rateLimit = require('express-rate-limit');
  const apiLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15分钟
    max: 100 // 每个IP限制100次请求
  });
  app.use('/api/', apiLimiter);
  ```
- **输出过滤**：防止 XSS 攻击

### 5. 静态与动态资源对比

| 特性           | 静态资源            | 动态资源               |
| -------------- | ------------------- | ---------------------- |
| **内容生成**   | 预先生成            | 实时生成               |
| **变化频率**   | 低                  | 高                     |
| **性能开销**   | 低                  | 高                     |
| **缓存能力**   | 强（长期缓存）      | 弱（短期/条件缓存）    |
| **个性化**     | 无                  | 有                     |
| **典型内容**   | HTML, CSS, JS, 图片 | API 响应, 用户数据     |
| **服务器处理** | 文件读取            | 业务逻辑+数据处理      |
| **CDN 友好度** | 优秀                | 有限（边缘计算可改善） |

### 6. 混合应用场景

#### 6.1 同构应用 (SSR)
```javascript
// Next.js 风格服务端渲染
app.get('/product/:id', async (req, res) => {
  const product = await Product.findById(req.params.id);
  
  // 渲染静态HTML框架
  const html = `
    <!DOCTYPE html>
    <html>
      <head>
        <title>${product.name}</title>
        <link rel="stylesheet" href="/static/css/product.css">
      </head>
      <body>
        <div id="root">${renderProduct(product)}</div>
        <script>
          // 注入初始数据
          window.__INITIAL_STATE__ = ${JSON.stringify({ product })};
        </script>
        <script src="/static/js/product.js"></script>
      </body>
    </html>
  `;
  
  res.send(html);
});
```

#### 6.2 API 网关 + 静态前端
```
客户端 → CDN（静态资源）
        ↓
客户端 → API 网关 → Node.js 微服务（动态资源）
```

#### 6.3 增量静态生成 (ISR)
```javascript
// Next.js 风格的增量静态再生
app.get('/blog/:slug', async (req, res) => {
  const slug = req.params.slug;
  const cachePath = path.join(__dirname, 'cache', `${slug}.html`);
  
  // 检查缓存是否存在且在有效期内
  if (fs.existsSync(cachePath) {
    const stats = fs.statSync(cachePath);
    if (Date.now() - stats.mtimeMs < 3600000) { // 1小时有效
      return res.sendFile(cachePath);
    }
  }
  
  // 动态生成内容
  const post = await BlogPost.findOne({ slug });
  const html = generatePostHtml(post);
  
  // 缓存新内容
  fs.writeFileSync(cachePath, html);
  res.send(html);
});
```

### 7. 最佳实践总结

#### 7.1 静态资源处理
1. 使用 CDN 分发全球加速
2. 设置长期缓存（1年+）
3. 启用压缩（Gzip/Brotli）
4. 实现内容指纹（`main.abc123.css`）
5. 使用 HTTP/2 提升加载效率

#### 7.2 动态资源处理

1. 实现分页和流式响应
2. 使用 Redis 缓存热点数据
3. 实施速率限制和 DDoS 防护
4. 数据库查询优化（索引、分片）
5. 异步处理耗时操作（队列）

#### 7.3 混合策略
1. 静态资源托管在 CDN/对象存储
2. 动态 API 使用 Node.js 微服务
3. 服务端渲染关键页面提升首屏性能
4. 使用边缘计算处理部分动态逻辑

通过合理区分和优化静态资源与动态资源的处理，可以显著提升 Node.js 应用的性能和用户体验。静态资源注重分发效率和缓存，动态资源关注实时性和个性化，二者结合构建现代 Web 应用的基石。



## 六、静态资源搭建

静态资源的搭建是 Web 开发的基础，合理规划目录结构和理解网站根目录的关系至关重要。下面我将从实际应用角度详细介绍静态资源的搭建方法和目录规划。

### 静态资源搭建的 4 种主要方式

#### 原生 Node.js 实现
```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');
const url = require('url');

const server = http.createServer((req, res) => {
  // 解析请求路径
  const parsedUrl = url.parse(req.url);
  let pathname = path.join(__dirname, 'public', parsedUrl.pathname);
  
  // 安全防护：防止目录遍历
  pathname = path.normalize(pathname).replace(/^(\.\.[\/\\])+/, '');
  
  // 检查文件是否存在
  fs.stat(pathname, (err, stats) => {
    if (err) {
      res.writeHead(404);
      res.end('File not found');
      return;
    }
    
    // 如果是目录，默认返回 index.html
    if (stats.isDirectory()) {
      pathname = path.join(pathname, 'index.html');
    }
    
    // 读取文件
    fs.readFile(pathname, (err, data) => {
      if (err) {
        res.writeHead(500);
        res.end('Server error');
        return;
      }
      
      // 根据扩展名设置 MIME 类型
      const ext = path.extname(pathname);
      const mimeTypes = {
        '.html': 'text/html',
        '.css': 'text/css',
        '.js': 'application/javascript',
        '.png': 'image/png',
        '.jpg': 'image/jpeg',
        '.gif': 'image/gif',
        '.svg': 'image/svg+xml'
      };
      
      res.setHeader('Content-Type', mimeTypes[ext] || 'application/octet-stream');
      res.end(data);
    });
  });
});

server.listen(3000, () => {
  console.log('Static server running on port 3000');
});
```

#### 使用 Express 框架
```javascript
const express = require('express');
const app = express();

// 基础静态资源服务
app.use(express.static('public'));

// 带虚拟路径的静态服务
app.use('/static', express.static('public'));

// 多个静态目录
app.use(express.static('public'));
app.use(express.static('uploads'));

// 带选项的静态服务
app.use('/assets', express.static('public', {
  index: false, // 禁用目录索引
  extensions: ['html', 'htm'], // 自动添加扩展名
  maxAge: '1d', // 浏览器缓存时间
  setHeaders: (res, path) => {
    if (path.endsWith('.css')) {
      res.set('Content-Encoding', 'gzip');
    }
  }
}));

app.listen(3000);
```

#### 使用 Koa 框架
```javascript
const Koa = require('koa');
const serve = require('koa-static');
const path = require('path');

const app = new Koa();

// 基本静态服务
app.use(serve('public'));

// 带选项的静态服务
app.use(serve('public', {
  maxage: 86400000, // 1天缓存
  hidden: false, // 不传输隐藏文件
  index: 'default.html', // 自定义索引文件
  gzip: true, // 自动gzip
  brotli: true // 支持brotli压缩
}));

app.listen(3000);
```

#### 专用静态资源服务器
```javascript
// 使用 node-static 模块
const static = require('node-static');
const http = require('http');

const file = new static.Server('./public', {
  cache: 3600, // 1小时缓存
  gzip: true,
  indexFile: 'index.html'
});

http.createServer((req, res) => {
  req.addListener('end', () => file.serve(req, res)).resume();
}).listen(8080);
```



## 七、静态资源目录与网站根目录的关系

### 1. 核心概念对比
| **概念**     | **静态资源目录**               | **网站根目录**                     |
| ------------ | ------------------------------ | ---------------------------------- |
| **定义**     | 存放静态文件的物理位置         | 网站的逻辑入口位置                 |
| **内容**     | HTML/CSS/JS/图片等文件         | 通常包含服务器脚本和入口文件       |
| **访问**     | 直接通过 URL 访问文件          | 通过域名直接访问的位置             |
| **示例路径** | `/var/www/site/public`         | `/var/www/site`                    |
| **典型结构** | 包含子目录：css/, js/, images/ | 包含：app.js, package.json, views/ |

### 2. 典型目录结构示例
```bash
website-root/         # 网站根目录
├── app.js            # Node.js 主入口文件
├── package.json      # 项目配置
├── node_modules/     # 依赖包
├── public/           # 静态资源目录（核心）
│   ├── index.html    # 首页
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── app.js
│   └── images/
│       └── logo.png
├── src/              # 服务器源代码
│   └── controllers/
├── views/            # 模板文件
└── .env              # 环境配置
```

### 3. 目录关系详解
1. **包含关系**：
   - 静态资源目录通常位于网站根目录内部（如 `/public`）
   - 网站根目录是项目的顶级目录

2. **访问映射**：
   - URL `https://example.com/css/style.css` 
   - → 映射到文件 `/public/css/style.css`

3. **安全隔离**：
   - 静态目录只包含可公开访问的文件
   - 敏感文件（如 `.env`、服务器代码）保留在根目录但不可公开访问

4. **部署优化**：
   ```nginx
   # Nginx 配置示例
   server {
       listen 80;
       server_name example.com;
       
       # 网站根目录处理动态请求
       location / {
           proxy_pass http://localhost:3000; # Node.js 应用
       }
       
       # 静态资源直接由 Nginx 处理
       location /static/ {
           alias /var/www/site/public/;
           expires 1y;
           add_header Cache-Control "public";
       }
   }
   ```

### 4. 静态资源目录最佳实践

#### 4.1 目录结构规范
```bash
public/
├── assets/           # 通用资源
│   ├── fonts/        # 字体文件
│   └── icons/        # 图标系统
├── css/              # 样式文件
│   ├── base.css
│   ├── components/
│   └── pages/
├── js/               # 脚本文件
│   ├── main.js       # 主入口
│   ├── lib/          # 第三方库
│   └── modules/      # ES模块
├── images/           # 图片资源
│   ├── backgrounds/
│   ├── icons/
│   └── products/
├── media/            # 媒体文件
│   ├── videos/
│   └── audio/
└── downloads/        # 可下载文件
```

#### 4.2 安全配置要点
```javascript
// Express 安全配置示例
app.use('/static', express.static('public', {
  dotfiles: 'ignore', // 忽略 .开头的文件
  setHeaders: (res, path) => {
    // 禁用 MIME 类型嗅探
    res.setHeader('X-Content-Type-Options', 'nosniff');
    
    // 防止点击劫持
    res.setHeader('X-Frame-Options', 'DENY');
    
    // 禁止目录列表
    res.setHeader('X-Directory-Listing', 'false');
    
    // 内容安全策略
    res.setHeader("Content-Security-Policy", "default-src 'self'");
    
    // 禁止缓存敏感文件
    if (path.endsWith('.html')) {
      res.setHeader('Cache-Control', 'no-store');
    }
  }
}));
```

#### 4.3 性能优化策略

##### 缓存控制
```javascript
// 缓存策略配置
app.use(express.static('public', {
  maxAge: '1y', // 长期缓存
  immutable: true, // 不可变资源
  etag: false, // 禁用ETag
  lastModified: false, // 禁用Last-Modified
  setHeaders: (res, path) => {
    // 带哈希的资源永久缓存
    if (path.match(/\.[0-9a-f]{8}\./)) {
      res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
    }
  }
}));
```

##### 压缩传输
```javascript
// 使用 compression 中间件
const compression = require('compression');
app.use(compression({
  level: 6, // 压缩级别
  threshold: 0, // 所有内容压缩
  filter: (req) => {
    // 排除已压缩的资源
    return !req.headers['x-no-compression'];
  }
}));
```

##### 内容分发网络 (CDN)
1. 将静态资源上传到 CDN
2. 修改资源引用地址：
```html
<!-- 本地开发 -->
<script src="/js/app.js"></script>

<!-- 生产环境 -->
<script src="https://cdn.example.com/js/app.abc123.js"></script>
```

### 5. 高级部署模式

#### 5.1 静态资源分离部署
```
独立静态资源服务器
  ↓
CDN 全球分发
  ↓
用户浏览器

Node.js 应用服务器 (只处理动态请求)
```

#### 5.2 自动化构建流水线
```mermaid
graph LR
A[源代码仓库] --> B[CI/CD 流水线]
B --> C{变更类型}
C -->|静态资源| D[构建优化]
C -->|服务器代码| E[重启服务]
D --> F[上传到CDN]
E --> G[部署到应用服务器]
```

#### 5.3 基于哈希的资源管理
```javascript
// Webpack 配置示例
output: {
  filename: '[name].[contenthash].js',
  path: path.resolve(__dirname, 'public/js'),
}

// 引用方式
<script src="/js/main.3c82a9b4.js"></script>
```

#### 5.4 混合渲染策略
```javascript
// Next.js 风格的混合渲染
app.get('/products/:id', (req, res) => {
  // 静态生成的产品页面
  const staticPage = path.join(__dirname, 'public', 'products', `${req.params.id}.html`);
  
  if (fs.existsSync(staticPage)) {
    // 存在静态版本则直接发送
    res.sendFile(staticPage);
  } else {
    // 否则动态生成
    const product = fetchProduct(req.params.id);
    res.render('product-detail', { product });
    
    // 后台生成静态页面
    generateStaticPage(product);
  }
});
```

### 6. 常见问题解决方案

#### 问题1：URL 路径冲突
**场景**：`/api/users` 和 `/static/api/users` 冲突  
**解决方案**：
```javascript
// 为API设置专用前缀
app.use('/api', apiRouter);

// 静态资源放在不同路径
app.use('/static', express.static('public'));
```

#### 问题2：单页应用路由
**配置**：
```javascript
// 所有未匹配路由返回 index.html
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});
```

#### 问题3：大型文件上传
**优化**：
```javascript
// 使用流式处理
app.post('/upload', (req, res) => {
  const fileStream = fs.createWriteStream('uploads/largefile.zip');
  req.pipe(fileStream);
  
  fileStream.on('finish', () => {
    res.status(201).send('Upload complete');
  });
});
```

### 7. 总结

静态资源搭建的关键要点：
1. **目录规划**：
   - 静态资源目录应独立组织（通常为 `/public`）
   - 网站根目录包含应用入口和配置文件

2. **部署策略**：
   - 开发环境：框架内置静态服务
   - 生产环境：Nginx 直接服务静态资源 + CDN 分发

3. **性能优化**：
   - 长期缓存：带哈希的资源设置 `max-age=1年`
   - 压缩传输：Gzip/Brotli 压缩
   - 资源合并：减少 HTTP 请求

4. **安全防护**：
   - 防止目录遍历
   - 禁用敏感文件访问
   - 设置安全头部

5. **现代实践**：
   - 静态资源与动态 API 分离部署
   - 自动化构建部署流水线
   - 混合渲染（SSG + SSR）

通过合理规划静态资源目录和正确理解其与网站根目录的关系，可以构建出高性能、易维护的 Web 应用。静态资源服务看似简单，但优化到位可显著提升用户体验和网站性能。



## 八、网页 URL 中的相对路径与绝对路径

在 Web 开发和 Node.js 应用中，理解路径的两种主要形式 - **相对路径**和**绝对路径** - 是构建健壮应用的基础。这两者有着根本性的区别和使用场景。

### 1. 核心概念对比

| **特性**         | 相对路径 (Relative Path) | 绝对路径 (Absolute Path)                  |
| ---------------- | ------------------------ | ----------------------------------------- |
| **定义**         | 相对于当前文档位置的路径 | 完整的网络资源地址                        |
| **起始标识**     | 不以协议开头             | 以协议开头（`http://`, `https://`, `//`） |
| **文件系统类比** | 类似 `./docs/file.txt`   | 类似 `/usr/local/docs/file.txt`           |
| **依赖关系**     | 依赖当前文档位置         | 独立于当前文档位置                        |
| **可移植性**     | 站点内移动文件时更灵活   | 位置固定，站点结构变化时需手动更新        |
| **典型使用场景** | 站点内部资源引用         | 跨站点引用、CDN 资源、规范链接            |

### 2. 相对路径详解

#### 2.1 基本语法形式

```html
<!-- 引用同级目录文件 -->
<img src="logo.png">

<!-- 引用子目录文件 -->
<script src="js/app.js"></script>

<!-- 引用上级目录文件 -->
<a href="../about.html">关于我们</a>

<!-- 引用根目录相对路径 -->
<link rel="stylesheet" href="/css/style.css">
```

#### 2.2 相对路径前缀解析
| **前缀** | 含义                     | 示例                 | 当前路径        | 解析结果           |
| -------- | ------------------------ | -------------------- | --------------- | ------------------ |
| (无)     | 同级目录                 | `image.jpg`          | /blog/post.html | /blog/image.jpg    |
| `./`     | 当前目录                 | `./config.json`      | /admin/         | /admin/config.json |
| `../`    | 父级目录                 | `../assets/logo.png` | /user/profile/  | /assets/logo.png   |
| `/`      | 根目录（服务器绝对路径） | `/images/banner.jpg` | 任意位置        | /images/banner.jpg |

#### 2.3 Node.js 中的相对路径处理

```javascript
const fs = require('fs');
const path = require('path');

// 危险：相对路径依赖执行位置
fs.readFile('config.json', (err, data) => {
  // 如果从不同目录执行可能出错
});

// 正确：使用 __dirname 转为绝对路径
const configPath = path.join(__dirname, 'config.json');
fs.readFile(configPath, (err, data) => {
  // 始终定位到正确文件
});

// Express 路由中的相对路径
app.get('/download', (req, res) => {
  // 使用绝对路径确保文件位置正确
  const filePath = path.resolve(__dirname, '../public/reports/report.pdf');
  res.download(filePath);
});
```

### 3. 绝对路径详解

#### 3.1 完整绝对路径结构
```
https://www.example.com:443/images/photo.jpg?size=large#section2
┌─┴─┐ ┌──────┬───────┐┌─┬─┐┌─────────────┬─────┐└─────┬─────┘└──┬──┘
协议  子域名   主域名   端口 路径          查询参数      锚点定位
```

#### 3.2 协议相关形式
| **类型**         | 示例                                | 说明                                    |
| ---------------- | ----------------------------------- | --------------------------------------- |
| **完整绝对路径** | `https://example.com/css/style.css` | 包含协议、域名和路径                    |
| **协议相对路径** | `//cdn.example.com/lib/jquery.js`   | 继承当前页面协议（HTTP/HTTPS 自动切换） |
| **根相对路径**   | `/images/logo.png`                  | 从网站根目录开始（非文件系统根目录）    |

#### 3.3 何时使用绝对路径

```html
<!-- 跨子域引用资源 -->
<script src="https://static.example.com/js/app.min.js"></script>

<!-- CDN 资源 -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">

<!-- 避免重复内容惩罚 (SEO) -->
<link rel="canonical" href="https://example.com/product/shoes">

<!-- 社交媒体分享预览 -->
<meta property="og:image" content="https://example.com/images/social-preview.jpg">
```

### 4. 路径选择决策指南

#### 4.1 相对路径适用场景
- 站点内部页面间的跳转链接
- 引用同服务器上的 CSS/JS 文件
- 项目内部模块的相互引用
- 需要随代码库一起移动的资源

#### 4.2 绝对路径适用场景
- 引用外部 CDN 上的资源
- 社交媒体分享和 SEO 优化
- 电子邮件中的资源链接
- 跨子域的资源共享
- RESTful API 的端点定义

#### 4.3 混合使用策略
```html
<head>
  <!-- 使用绝对路径确保外部资源始终可访问 -->
  <link rel="stylesheet" href="https://cdn.example.com/framework.css">
  
  <!-- 相对路径引用站点自有资源 -->
  <link rel="stylesheet" href="/css/main.css">
  <script src="/js/app.js"></script>
  
  <!-- 协议相对路径用于第三方服务 -->
  <script src="//maps.googleapis.com/maps/api/js?key=API_KEY"></script>
</head>
```

### 5. Node.js 中的路径处理实践

#### 5.1 路径解析工具
```javascript
const path = require('path');

// 解析为绝对路径
console.log(path.resolve('src', 'app.js')); 
// 输出: /projects/myapp/src/app.js

// 路径规范化
console.log(path.normalize('/foo/bar//baz/../qux')); 
// 输出: /foo/bar/qux

// 获取文件扩展名
console.log(path.extname('index.html')); 
// 输出: .html
```

#### 5.2 URL 处理模块
```javascript
const url = require('url');

const website = 'https://example.com:8080/path/page.html?query=string#hash';

// 解析 URL
const parsedUrl = new URL(website);
console.log(parsedUrl.protocol); // 'https:'
console.log(parsedUrl.host);     // 'example.com:8080'
console.log(parsedUrl.pathname); // '/path/page.html'
console.log(parsedUrl.search);   // '?query=string'
console.log(parsedUrl.hash);     // '#hash'

// 构建 URL
const newUrl = new URL('/new-path', 'https://example.com');
console.log(newUrl.href); // 'https://example.com/new-path'
```

#### 5.3 在 Express 中处理路径
```javascript
const express = require('express');
const app = express();

// 提供静态文件服务（绝对路径更安全）
app.use('/static', express.static(path.join(__dirname, 'public')));

// 重定向使用绝对路径
app.get('/old-page', (req, res) => {
  res.redirect(301, '/new-page');
});

// 生成绝对 URL
app.get('/generate-link', (req, res) => {
  const fullUrl = `${req.protocol}://${req.get('host')}/download`;
  res.send(`下载地址: <a href="${fullUrl}">点击下载</a>`);
});
```

### 6. 常见问题解决方案

#### 6.1 相对路径失效问题
**场景**：`<a href="about.html">` 在 `/blog/post.html` 页面点击后跳转到 `/blog/about.html` 而非预期的 `/about.html`  
**解决**：
```html
<!-- 使用根相对路径 -->
<a href="/about.html">关于我们</a>
```

#### 6.2 混合内容警告 (Mixed Content)
**场景**：HTTPS 页面加载了 HTTP 资源  
**解决**：
```html
<!-- 使用协议相对路径 -->
<script src="//cdn.example.com/library.js"></script>
```

#### 6.3 Node.js 中的路径困惑
**场景**：`fs.readFile('file.txt')` 在不同目录执行时行为不一致  
**解决**：
```javascript
// 始终使用 __dirname 构建绝对路径
const filePath = path.join(__dirname, 'data', 'file.txt');
fs.readFile(filePath, 'utf8', (err, data) => { ... });
```

#### 6.4 单页应用路由问题
**场景**：刷新页面或直接访问子路由时 404  
**解决**：
```javascript
// Express 配置
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});
```

### 7. 路径最佳实践

1. **前端开发**：
   - 站点内部链接使用相对路径
   - 关键资源（CSS/JS）使用根相对路径（`/css/main.css`）
   - 外部资源使用 HTTPS 绝对路径

2. **Node.js 后端**：
   - 所有文件路径使用 `path.join()` 和 `__dirname`
   - 使用 `path.resolve()` 确保绝对路径
   - 避免在配置中使用相对路径

3. **SEO 优化**：
   - 规范 URL 使用绝对路径
   - 面包屑导航使用结构化数据
   - 确保所有内部链接一致（避免 http/https 混用）

4. **安全考虑**：
   - 验证用户提供的路径，防止目录遍历攻击
   ```javascript
   function safePath(baseDir, userInput) {
     const fullPath = path.resolve(baseDir, userInput);
     if (!fullPath.startsWith(path.resolve(baseDir))) {
       throw new Error('非法路径访问');
     }
     return fullPath;
   }
   ```

5. **跨平台兼容**：
   ```javascript
   // 使用 path.sep 代替硬编码斜杠
   const filePath = `data${path.sep}users${path.sep}profile.json`;
   
   // 在 URL 中始终使用正斜杠 /
   const apiUrl = '/api/v2/users';
   ```

### 8. 总结

在 Web 开发和 Node.js 应用中：

1. **相对路径**是上下文相关的路径，适合：
   - 站点内部资源引用
   - 与当前文件位置相关的资源
   - 需要随项目迁移的资源

2. **绝对路径**是完整确定的路径，适合：
   - 跨域资源引用
   - CDN 资源加载
   - 规范链接和社交媒体分享
   - 需要精确位置的服务器操作

**黄金法则**：
- 在前端 HTML 中，优先使用相对路径和根相对路径
- 在 Node.js 后端代码中，始终将路径转为绝对路径
- 对外部资源始终使用 HTTPS 绝对路径
- 使用 `path` 和 `url` 模块处理路径，避免手动拼接

掌握路径的正确使用方式，可以避免许多常见的开发陷阱，构建出更健壮、更可维护的 Web 应用。



## 九、MIME 

**MIME（Multipurpose Internet Mail Extensions）类型**是互联网上标识文件格式的标准方法，它像文件的"身份证"一样告诉浏览器或应用程序如何处理接收到的内容。最初为电子邮件设计，现已成为 Web 通信的基石。

### 1. 核心概念解析

#### 1.1 MIME 类型结构
```http
Content-Type: type/subtype; parameter=value
```
- **主类型（Type）**：数据的大类
- **子类型（Subtype）**：具体的格式类型
- **参数（可选）**：如字符集等额外信息

**示例**：
```http
Content-Type: text/html; charset=utf-8
Content-Type: image/png
Content-Type: application/json
```

#### 1.2 常见 MIME 类型分类
| 主类型        | 子类型示例              | 典型文件格式        | 使用场景                 |
| ------------- | ----------------------- | ------------------- | ------------------------ |
| `text`        | html, plain, css, csv   | .html, .txt, .css   | 网页、样式表、纯文本     |
| `image`       | jpeg, png, gif, svg+xml | .jpg, .png, .gif    | 图片展示                 |
| `audio`       | mpeg, ogg, wav, webm    | .mp3, .ogg, .wav    | 音频播放                 |
| `video`       | mp4, ogg, webm          | .mp4, .ogv, .webm   | 视频播放                 |
| `application` | json, pdf, xml, zip     | .json, .pdf, .zip   | 数据交换、文档、压缩文件 |
| `multipart`   | mixed, alternative      | 电子邮件附件        | 多部分内容组合           |
| `font`        | woff, woff2, ttf        | .woff, .woff2, .ttf | 网页字体                 |

### 2. Web 开发中的关键作用

#### 2.1 浏览器行为控制
```http
# 触发下载行为
Content-Type: application/octet-stream
Content-Disposition: attachment; filename="report.pdf"

# 直接显示内容
Content-Type: text/html
```

#### 2.2 文件处理差异对比
| MIME 类型          | 浏览器行为         | Node.js 处理方式    |
| ------------------ | ------------------ | ------------------- |
| `text/html`        | 渲染为网页         | 无需特殊处理        |
| `application/json` | 显示格式化 JSON    | `JSON.parse()` 解析 |
| `image/png`        | 显示图片           | 二进制流处理        |
| `application/pdf`  | 内置或插件打开 PDF | 使用 PDF 解析库     |

#### 2.3 Node.js 中的 MIME 类型应用
```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');

http.createServer((req, res) => {
  const filePath = path.join(__dirname, 'static', req.url);
  const extname = path.extname(filePath);
  
  // MIME 类型映射
  const mimeTypes = {
    '.html': 'text/html',
    '.js': 'text/javascript',
    '.css': 'text/css',
    '.json': 'application/json',
    '.png': 'image/png',
    '.jpg': 'image/jpeg',
    '.gif': 'image/gif',
    '.ico': 'image/x-icon',
    '.svg': 'image/svg+xml',
    '.pdf': 'application/pdf'
  };

  // 设置 Content-Type
  const contentType = mimeTypes[extname] || 'application/octet-stream';
  
  fs.readFile(filePath, (err, data) => {
    if (err) {
      res.writeHead(404);
      res.end('File not found');
    } else {
      res.writeHead(200, { 'Content-Type': contentType });
      res.end(data);
    }
  });
}).listen(8080);
```

### 3. 特殊 MIME 类型详解

#### 3.1 通用类型
```http
application/octet-stream   # 未知二进制文件（默认下载）
application/x-www-form-urlencoded # 表单提交数据
multipart/form-data        # 包含文件上传的表单
```

#### 3.2 现代 Web 类型
```http
application/wasm           # WebAssembly 模块
application/manifest+json  # Web App Manifest
font/woff2                # WOFF2 字体格式
image/webp                # WebP 图片格式
```

#### 3.3 易混淆类型对比
| 文件类型   | 正确 MIME                | 常见错误用法      | 后果                   |
| ---------- | ------------------------ | ----------------- | ---------------------- |
| JavaScript | `application/javascript` | `text/javascript` | 老浏览器兼容，但非标准 |
| XML        | `application/xml`        | `text/xml`        | 部分场景可工作但非最佳 |
| JSON       | `application/json`       | `text/json`       | 非标准，可能被拒绝     |
| SVG        | `image/svg+xml`          | `image/svg`       | 无法正确渲染           |

### 4. MIME 嗅探与安全防护

#### 4.1 浏览器 MIME 嗅探风险
当服务器发送错误的 Content-Type 时，浏览器可能：
- 将文本文件当作 HTML 执行（XSS 攻击）
- 将图片当作可执行脚本处理

#### 4.2 防护措施
```http
# 禁止浏览器嗅探
X-Content-Type-Options: nosniff

# Express 设置示例
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  next();
});
```

#### 4.3 内容安全策略（CSP）
```http
Content-Security-Policy: default-src 'self'
```

### 5. 实践应用场景

#### 5.1 文件上传类型验证
```javascript
// Express 文件上传校验
const upload = multer({
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
    if (!allowedTypes.includes(file.mimetype)) {
      return cb(new Error('Invalid file type'), false);
    }
    cb(null, true);
  }
});
```

#### 5.2 动态内容生成
```javascript
// 生成并返回 CSV
app.get('/export', (req, res) => {
  const csvData = 'Name,Email\nJohn,john@example.com';
  res.header('Content-Type', 'text/csv');
  res.attachment('users.csv');
  res.send(csvData);
});
```

#### 5.3 正确服务 Web 字体
```css
/* CSS 中声明 */
@font-face {
  font-family: 'MyFont';
  src: url('font.woff2') format('woff2'),
       url('font.woff') format('woff');
}
```

### 6. MIME 类型检测工具

#### 6.1 Node.js 自动检测
```javascript
const mime = require('mime-types');

console.log(mime.contentType('markdown.md')); // 'text/markdown'
console.log(mime.extension('application/json')); // 'json'
```

#### 6.2 文件魔术数字检测
```javascript
const fileType = require('file-type');

async function detectFile(buffer) {
  const type = await fileType.fromBuffer(buffer);
  console.log(type); // { ext: 'png', mime: 'image/png' }
}
```

### 7. 最佳实践总结

1. **始终设置 Content-Type**：
   - 静态文件服务明确类型
   - API 响应使用 `application/json`

2. **标准化类型使用**：
   - 优先使用 `application/javascript` 而非 `text/javascript`
   - JSON 数据使用 `application/json`

3. **安全防护**：
   - 强制设置 `X-Content-Type-Options: nosniff`
   - 上传文件验证 MIME 类型

4. **字符集声明**：
   ```http
   Content-Type: text/html; charset=utf-8
   ```

5. **现代格式支持**：
   - 图像优先使用 `image/webp`
   - 字体使用 `font/woff2`

> **关键点**：MIME 类型是 Web 通信的"语言翻译官"，正确设置可确保：
>
> - 浏览器正确渲染内容
> - 应用程序准确解析数据
> - 有效防御内容类型攻击

理解并正确使用 MIME 类型，是构建健壮 Web 应用的基础技能，尤其在处理文件上传下载、API 设计和静态资源服务时至关重要。



## 十、模块导入

Node.js 的模块导入系统遵循一套明确的规则，理解这些规则对构建可维护的应用程序至关重要。下面我将全面解析 Node.js 导入文件和文件夹的规则体系。

### 1. 核心导入机制：require() 和 import

#### 1.1 CommonJS (require) 导入规则
```javascript
// 导入核心模块
const fs = require('fs');

// 导入本地文件
const utils = require('./utils');

// 导入目录
const config = require('./config');
```

#### 1.2 ES Modules (import) 导入规则
```javascript
// 导入核心模块
import fs from 'fs';

// 导入本地文件
import utils from './utils.js'; // 注意扩展名要求

// 导入目录
import config from './config/index.js';
```

### 2. 文件导入规则

#### 2.1 文件路径解析顺序
Node.js 解析文件路径的优先级：

1. **精确路径匹配**：优先查找指定路径
   ```javascript
   require('./utils.js') // 直接加载 utils.js
   ```

2. **扩展名自动补全**（仅 CommonJS）：
   ```javascript
   require('./utils') // 依次尝试：
   // 1. utils.js
   // 2. utils.json
   // 3. utils.node (C++插件)
   ```

3. **目录模块解析**（当路径是目录时）

#### 2.2 文件类型处理
| **文件类型** | **处理方式**             | **示例**                |
| ------------ | ------------------------ | ----------------------- |
| `.js`        | 作为 JavaScript 模块执行 | `require('./app')`      |
| `.json`      | 解析为 JSON 对象         | `require('./config')`   |
| `.node`      | 加载编译的 C++ 插件      | `require('bcrypt')`     |
| `.mjs`       | 作为 ES 模块处理         | `import './module.mjs'` |
| `.cjs`       | 作为 CommonJS 模块处理   | `require('./file.cjs')` |

### 3. 文件夹导入规则

#### 3.1 目录解析优先级
当路径指向目录时，Node.js 按以下顺序查找入口：

1. **package.json 的 main 字段**
   ```json
   // my-module/package.json
   {
     "main": "lib/main.js"
   }
   ```
   ```javascript
   require('./my-module') // 加载 my-module/lib/main.js
   ```

2. **index 文件**（按扩展名顺序）
   ```
   index.js
   index.json
   index.node
   ```

3. **失败**：抛出 `MODULE_NOT_FOUND` 错误

#### 3.2 目录结构示例
```
my-package/
├── package.json  // 包含 "main": "dist/index.js"
├── src/
│   └── index.js
└── dist/
    ├── index.js
    └── utils.js
```

```javascript
// 导入将指向 dist/index.js
const myPackage = require('./my-package');
```

### 4. 模块解析全流程

#### 4.1 require() 解析算法
```mermaid
graph TD
    A[require_module] --> B{是核心模块?}
    B -->|是| C[加载核心模块]
    B -->|否| D{以 './' 或 '/' 开头?}
    D -->|是| E[解析为文件路径]
    D -->|否| F[在node_modules查找]
    E --> G{路径存在?}
    G -->|是| H[加载文件/目录]
    G -->|否| I[尝试添加扩展名]
    I --> J{找到文件?}
    J -->|是| H
    J -->|否| K[抛出错误]
    F --> L[当前目录node_modules]
    L --> M{找到?}
    M -->|是| H
    M -->|否| N[父目录node_modules]
    N --> O{找到?}
    O -->|是| H
    O -->|否| P[继续向上查找]
    P --> Q{到根目录?}
    Q -->|是| K
```

#### 4.2 实际解析示例
```javascript
// 项目结构:
// project/
//   ├── src/
//   │   └── app.js
//   ├── lib/
//   │   ├── utils.js
//   │   └── helpers/
//   │       ├── index.js
//   │       └── math.js
//   └── node_modules/
//       └── lodash/

// app.js 中的导入:
require('./lib/utils'); // 文件: lib/utils.js
require('./lib/helpers'); // 目录: lib/helpers/index.js
require('lodash'); // 模块: node_modules/lodash
```

### 5. node_modules 解析规则

#### 5.1 查找算法
Node.js 从当前目录开始向上递归查找：
1. `/project/src/node_modules`
2. `/project/node_modules`
3. `/node_modules`
4. 直到文件系统根目录

#### 5.2 模块加载优先级
```javascript
require('module') // 查找顺序:
1. 核心模块
2. 当前目录/node_modules/module
3. 父目录/node_modules/module
4. 继续向上...
```

#### 5.3 子目录优先原则
```javascript
// 优先查找 module 子目录
require('module/component') 
// 解析为: node_modules/module/component.js
// 或: node_modules/module/component/index.js
```

### 6. ES Modules 特殊规则

#### 6.1 扩展名要求
```javascript
// 必须包含文件扩展名
import utils from './utils.js'; // ✅ 正确
import helper from './helper'; // ❌ 错误（除非配置实验性标志）
```

#### 6.2 目录导入差异
```javascript
// 必须明确指定目录入口
import config from './config/index.js'; // ✅ 正确
import config from './config'; // ❌ 错误
```

#### 6.3 启用 ESM 的方式
1. **文件扩展名**：使用 `.mjs`
2. **package.json**：
   ```json
   {
     "type": "module"
   }
   ```
3. **命令行标志**：`node --input-type=module`

### 7. 路径解析辅助工具

#### 7.1 获取绝对路径
```javascript
const path = require('path');

// 获取当前文件所在目录
console.log(__dirname); // /project/src

// 解析为绝对路径
const fullPath = path.resolve('config', 'app.json');
console.log(fullPath); // /project/config/app.json
```

#### 7.2 模块路径解析
```javascript
// 解析模块的完整路径
console.log(require.resolve('lodash')); 
// /project/node_modules/lodash/index.js

// 检查模块是否存在
require.resolve('non-existing-module'); // 抛出错误
```

### 8. 高级场景与解决方案

#### 8.1 循环依赖处理
```javascript
// a.js
exports.loaded = false;
const b = require('./b');
console.log('在 a 中, b.loaded =', b.loaded); // false
exports.loaded = true;

// b.js
exports.loaded = false;
const a = require('./a');
console.log('在 b 中, a.loaded =', a.loaded); // false
exports.loaded = true;
```

#### 8.2 动态导入
```javascript
// CommonJS
const loadModule = (name) => require(name);

// ES Modules
const loadModule = async (name) => {
  const module = await import(name);
  return module.default;
};
```

#### 8.3 条件导入
```javascript
// 根据环境导入不同模块
let dbModule;
if (process.env.NODE_ENV === 'test') {
  dbModule = require('./mock-db');
} else {
  dbModule = require('./real-db');
}
```

#### 8.4 模块缓存管理
```javascript
// 清除模块缓存
function clearModuleCache(modulePath) {
  const resolvedPath = require.resolve(modulePath);
  delete require.cache[resolvedPath];
}

// 热重载模块
const watcher = require('chokidar').watch('./modules');
watcher.on('change', (path) => {
  clearModuleCache(path);
  console.log('模块已更新:', path);
});
```

### 9. 最佳实践指南

1. **路径规范**：
   ```javascript
   // 使用 path.join 代替手动拼接
   const configPath = path.join(__dirname, 'config', 'app.json');
   ```

2. **模块组织**：
   ```
   project/
   ├── lib/
   │   ├── utils.js        // 工具函数
   │   └── services/       // 服务模块
   │       ├── user.js
   │       └── product.js
   ├── config/
   │   └── index.js        // 目录入口
   └── app.js
   ```

3. **导入风格**：
   ```javascript
   // 明确导入所需内容
   const { formatDate, parseDate } = require('./date-utils');
   
   // 避免通配符导入
   // ❌ 避免: const utils = require('./utils');
   ```

4. **ESM 迁移策略**：
   - 新项目：直接使用 `"type": "module"`
   - 旧项目：逐步迁移为 `.cjs` 和 `.mjs` 扩展名

### 10. 常见错误解决方案

#### 10.1 模块未找到
```bash
Error: Cannot find module './utils'
```
**解决**：
- 检查文件路径和大小写
- 确认文件扩展名（ESM 必须明确扩展名）
- 使用 `path.resolve` 确保正确路径

#### 10.2 目录导入失败
```bash
Error [ERR_UNSUPPORTED_DIR_IMPORT]: Directory import not supported
```
**解决**：
- 明确指定目录入口文件：`import './config/index.js'`
- 添加 `package.json` 定义 `"main"` 字段

#### 10.3 模块初始化错误
```bash
TypeError: myModule.function is not a function
```
**解决**：
- 检查模块导出是否正确
- 确认没有循环依赖问题
- 清除模块缓存后重试

### 11. 总结：Node.js 导入规则要点

1. **文件导入**：
   - CommonJS：自动补全扩展名（.js > .json > .node）
   - ESM：必须明确扩展名

2. **目录导入**：
   - 查找 `package.json` 的 `main` 字段
   - 查找 `index.js`/`index.json`/`index.node`

3. **模块解析**：
   - 核心模块 > 相对路径 > node_modules
   - node_modules 递归向上查找

4. **路径处理**：
   - 使用 `__dirname` 和 `path` 模块处理路径
   - 优先使用绝对路径

5. **现代实践**：
   - 新项目首选 ES Modules
   - 合理组织目录结构
   - 明确导出和导入内容

理解并遵循这些规则，将帮助您构建更健壮、可维护的 Node.js 应用程序，避免常见的模块解析错误和路径问题。



## 十一、npm 

npm (Node Package Manager) 是 Node.js 生态的核心工具，也是全球最大的软件注册中心。理解 npm 的工作原理和最佳实践是 Node.js 开发的必备技能。

### 1. npm 架构全景图

```bash
┌──────────────────────┐
│      npm 注册中心      │
│  (registry.npmjs.org) │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│    本地 npm 环境       │
├──────────────────────┤
│  package.json        │ ◀── 项目配置清单
│  package-lock.json   │ ◀── 精确依赖树
│  node_modules/       │ ◀── 依赖存储目录
│  npm-cache/          │ ◀── 下载包缓存
└──────────────────────┘
```

### 2. 核心概念详解

#### 2.1 package.json - 项目身份证
```json
{
  "name": "my-project",         // 包名（必须全小写无空格）
  "version": "1.0.0",           // 语义化版本
  "description": "项目描述",
  "main": "index.js",           // 入口文件
  "scripts": {                  // 自定义脚本
    "start": "node index.js",
    "test": "jest"
  },
  "dependencies": {             // 生产依赖
    "express": "^4.18.2"
  },
  "devDependencies": {          // 开发依赖
    "eslint": "^8.45.0"
  },
  "peerDependencies": {         // 宿主环境依赖
    "react": ">=16.8.0"
  },
  "engines": {                  // 环境要求
    "node": ">=18.0.0"
  }
}
```

#### 2.2 版本控制规则（语义化版本）

```
主版本号.次版本号.修订号
^4.18.2  →  4.x.x  (自动更新次版本和修订号)
~4.18.2  →  4.18.x (仅自动更新修订号)
4.18.2   →  严格锁定版本
```

#### 2.3 依赖类型对比
| **依赖类型**         | 安装命令               | 打包包含 | 使用场景           |
| -------------------- | ---------------------- | -------- | ------------------ |
| dependencies         | `npm install <pkg>`    | 是       | 生产环境必需依赖   |
| devDependencies      | `npm install -D <pkg>` | 否       | 开发/测试工具      |
| peerDependencies     | 不自动安装             | 否       | 插件类库的宿主依赖 |
| optionalDependencies | `npm install`          | 尝试安装 | 非必需依赖         |
| bundledDependencies  | 手动指定               | 是       | 需一起发布的依赖   |

### 3. npm 核心工作流程

#### 3.1 依赖安装算法
```mermaid
graph TD
    A[读取 package.json] --> B[检查 package-lock.json]
    B -->|存在| C[根据 lock 文件精确安装]
    B -->|不存在| D[解析依赖树]
    D --> E[下载依赖包]
    E --> F[解压到 node_modules]
    F --> G[执行 install 钩子]
    G --> H[生成/更新 package-lock.json]
```

#### 3.2 node_modules 结构演进
- **嵌套结构** (npm v2)
  ```
  node_modules/
  ├── moduleA/
  │   └── node_modules/
  │       ├── moduleC@1.0.0/
  ├── moduleB/
  │   └── node_modules/
  │       ├── moduleC@2.0.0/
  ```
  
- **扁平结构** (npm v3+)
  ```
  node_modules/
  ├── moduleA/
  ├── moduleB/
  ├── moduleC@1.0.0/  // 被 moduleA 使用
  └── moduleC@2.0.0/  // 被 moduleB 使用
  ```

- **安全锁定** (npm v5+)
  - package-lock.json 确保依赖树一致性
  - 优先使用扁平结构，冲突时嵌套

### 4. 核心命令详解

#### 4.1 依赖管理
```bash
# 安装项目所有依赖
npm install

# 安装生产依赖
npm install express

# 安装开发依赖
npm install --save-dev eslint

# 更新依赖
npm update

# 卸载依赖
npm uninstall express
```

#### 4.2 脚本执行
```bash
# 运行自定义脚本
npm run test

# 快捷命令（start/test/restart/stop）
npm start

# 并行运行脚本
npm install -g npm-run-all
run-p build:*
```

#### 4.3 包发布
```bash
# 登录 npm 账户
npm login

# 发布包
npm publish

# 撤销发布（24小时内）
npm unpublish <package>@<version>
```

#### 4.4 安全审计
```bash
# 检查漏洞
npm audit

# 自动修复
npm audit fix

# 查看依赖树
npm list --depth=2
```

### 5. 高级功能与技巧

#### 5.1 npx - 包执行器
```bash
# 临时安装并执行包
npx create-react-app my-app

# 执行本地包
npx eslint src/

# 执行不同版本的包
npx node@18 my-script.js
```

#### 5.2 工作区管理 (Workspaces)
```json
// package.json
{
  "name": "monorepo",
  "workspaces": ["packages/*"],
  "private": true
}
```
```bash
# 安装所有工作区依赖
npm install

# 为特定工作区安装依赖
npm install lodash -w frontend
```

#### 5.3 依赖覆盖与别名
```json
{
  "dependencies": {
    "moment": "npm:moment-timezone@0.5.43"
  },
  "overrides": {
    "react": "18.2.0"
  }
}
```

#### 5.4 自定义脚本钩子
```json
{
  "scripts": {
    "preinstall": "echo '开始安装...'",
    "postinstall": "node ./scripts/setup.js",
    "prepublish": "npm run build"
  }
}
```

### 6. 性能优化策略

#### 6.1 依赖安装加速
```bash
# 使用缓存优先策略
npm install --prefer-offline

# 跳过可选依赖
npm install --no-optional

# 全局安装常用工具
npm install -g typescript eslint prettier
```

#### 6.2 CI/CD 优化
```bash
# 仅安装生产依赖
npm install --production

# 缓存 node_modules
cache:
  directories:
    - node_modules

# 并行安装
npm install --legacy-peer-deps & npm run build
```

#### 6.3 依赖清理
```bash
# 检测未使用依赖
npm install -g depcheck
depcheck

# 删除重复依赖
npm dedupe

# 清理缓存
npm cache clean --force
```

### 7. 企业级最佳实践

#### 7.1 依赖安全策略
```bash
# 使用 npm ci 替代 install
npm ci # 严格按 lock 文件安装

# 锁定依赖版本
{
  "dependencies": {
    "lodash": "4.17.21" # 精确版本
  }
}
```

#### 7.2 私有注册源配置
```bash
# 设置公司私有源
npm config set registry https://registry.my-company.com

# 使用认证令牌
npm config set //registry.my-company.com/:_authToken TOKEN

# 作用域包管理
npm install @mycompany/logger
```

#### 7.3 多阶段 Docker 构建
```dockerfile
# 构建阶段
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 生产阶段
FROM node:18-alpine
COPY --from=builder /app/dist /app
COPY --from=builder /app/node_modules /app/node_modules
CMD ["node", "/app/index.js"]
```

### 8. 常见问题解决方案

#### 8.1 依赖冲突
**症状**：`ERESOLVE unable to resolve dependency tree`  
**解决**：
```bash
# 查看依赖树
npm list --depth=5

# 手动安装兼容版本
npm install react@18 react-dom@18

# 强制安装（最后手段）
npm install --legacy-peer-deps
```

#### 8.2 安装超时
**解决**：
```bash
# 切换淘宝源
npm config set registry https://registry.npmmirror.com

# 增加超时时间
npm install --fetch-timeout=600000

# 使用持久连接
npm config set fetch-retry-mintimeout 20000
npm config set fetch-retry-maxtimeout 120000
```

#### 8.3 磁盘空间不足
**优化**：
```bash
# 清除缓存
npm cache clean --force

# 删除重复文件
npx npm-dedupe

# 使用符号链接
npm install --global-style
```

### 9. npm 生态发展趋势

#### 9.1 现代包管理工具对比
| **特性**       | npm  | Yarn   | pnpm           |
| -------------- | ---- | ------ | -------------- |
| 安装速度       | 中等 | 快     | 极快           |
| 磁盘空间       | 高   | 高     | 低（符号链接） |
| 安全审计       | 完善 | 完善   | 完善           |
| Workspace 支持 | 支持 | 优秀   | 优秀           |
| 兼容性         | 原生 | 需适配 | 需适配         |

#### 9.2 npm v9+ 新特性
- **依赖隔离增强**：减少幻影依赖
- **Workspace 优化**：`npm workspaces` 命令集
- **安全改进**：自动签名验证
- **ESM 支持**：纯 ES Module 包管理

#### 9.3 未来发展方向
- **零安装架构**：类似 Deno 的远程依赖管理
- **WebAssembly 包**：原生支持 wasm 模块
- **区块链注册中心**：去中心化包分发
- **AI 辅助依赖**：智能依赖推荐和漏洞预测

### 10. npm 命令速查表

| **命令**              | **功能**              |
| --------------------- | --------------------- |
| `npm init`            | 创建 package.json     |
| `npm install`         | 安装所有依赖          |
| `npm install <pkg>`   | 安装特定包            |
| `npm update`          | 更新所有依赖          |
| `npm outdated`        | 检查过时依赖          |
| `npm list --depth=0`  | 查看顶层依赖          |
| `npm run <script>`    | 运行自定义脚本        |
| `npm publish`         | 发布包                |
| `npm deprecate <pkg>` | 标记弃用包            |
| `npm doctor`          | 检查 npm 环境健康状况 |
| `npm exec -- <cmd>`   | 执行本地安装的命令    |
| `npm explain <pkg>`   | 解释包被安装的原因    |

### 11. 总结：npm 核心价值与实践原则

1. **依赖管理黄金法则**：
   - 始终提交 `package-lock.json` 到版本控制
   - 在 CI 环境中使用 `npm ci` 而非 `npm install`
   - 定期执行 `npm audit` 检查安全漏洞

2. **版本控制最佳实践**：
   ```json
   {
     "dependencies": {
       "library": "^1.2.3", // 允许自动更新次版本和补丁
       "critical": "1.2.3"  // 关键依赖固定版本
     }
   }
   ```

3. **性能优化优先**：
   - 使用 `npm install --omit=dev` 生产环境安装
   - 配置 `.npmrc` 使用国内镜像源
   - 采用 pnpm 或 Yarn 提升大型项目性能

4. **安全防护措施**：
   ```bash
   # 预提交检查
   npx husky add .husky/pre-commit "npm audit"
   
   # 自动更新
   npx npm-check-updates -u
   npm install
   ```

npm 作为 Node.js 生态的基石，掌握其深度用法不仅能提升开发效率，更能构建出安全、稳定的应用程序。随着现代前端工程化的发展，npm 已从简单的包管理工具演变为完整的开发生命周期管理平台。



## 十二、Express

Express.js 是 Node.js 生态系统中最流行、最广泛使用的 Web 应用框架，它提供了强大的功能来构建 Web 应用、API 服务和微服务。自 2010 年发布以来，Express 已成为 Node.js Web 开发的事实标准。

### 1. Express 核心架构设计

```bash
┌──────────────────────┐
│      应用程序层        │
├──────────────────────┤
│      路由系统         │  ◀── 处理 HTTP 请求路由
├──────────────────────┤
│    中间件管道         │  ◀── 请求处理流水线
│  ┌─┬─┬─┬─┬─┐         │
│  │ │ │ │ │ │         │
│  └─┴─┴─┴─┴─┘         │
├──────────────────────┤
│ HTTP 请求/响应扩展    │  ◀── 增强 req/res 对象
├──────────────────────┤
│  底层 Node.js HTTP   │
└──────────────────────┘
```

### 2. 核心特性详解

#### 2.1 极简启动示例
```javascript
const express = require('express');
const app = express();
const port = 3000;

// 基本路由
app.get('/', (req, res) => {
  res.send('Hello Express!');
});

// 启动服务器
app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

#### 2.2 路由系统（Routing）
```javascript
// 基础路由
app.get('/products', (req, res) => {
  res.json([{id: 1, name: 'Product A'}]);
});

// 路由参数
app.get('/users/:userId', (req, res) => {
  res.send(`User ID: ${req.params.userId}`);
});

// 路由链式调用
app.route('/books')
  .get((req, res) => res.send('Get all books'))
  .post((req, res) => res.send('Add a book'));

// 路由模块化
const router = express.Router();
router.get('/', (req, res) => res.send('Blog Home'));
router.get('/post/:id', (req, res) => res.send(`Blog Post ${req.params.id}`));

app.use('/blog', router);
```

#### 2.3 中间件（Middleware）机制
```javascript
// 应用级中间件
app.use((req, res, next) => {
  console.log(`Request: ${req.method} ${req.path}`);
  next(); // 传递给下一个中间件
});

// 路由级中间件
const authMiddleware = (req, res, next) => {
  if (req.headers.authorization) {
    next();
  } else {
    res.status(401).send('Unauthorized');
  }
};

app.get('/admin', authMiddleware, (req, res) => {
  res.send('Admin Dashboard');
});

// 错误处理中间件
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});

// 第三方中间件
const bodyParser = require('body-parser');
app.use(bodyParser.json()); // 解析 JSON 请求体
```

#### 2.4 请求与响应对象扩展
```javascript
// 请求对象增强
app.get('/search', (req, res) => {
  console.log(req.query);    // 获取查询参数 ?q=express
  console.log(req.cookies);  // 获取 cookies（需 cookie-parser）
  console.log(req.ip);       // 客户端 IP
});

// 响应对象方法
app.post('/data', (req, res) => {
  res.status(201).json({ id: 123 }); // 设置状态码 + JSON
  res.cookie('token', 'abc123');     // 设置 cookie
  res.set('X-Custom-Header', 'value'); // 设置响应头
  res.redirect('/success');          // 重定向
});
```

### 3. 核心中间件详解

#### 3.1 官方维护中间件
| **中间件**           | **功能**         | **安装**           |
| -------------------- | ---------------- | ------------------ |
| express.json()       | 解析 JSON 请求体 | Express 4.16+ 内置 |
| express.urlencoded() | 解析表单数据     | Express 4.16+ 内置 |
| express.static()     | 提供静态文件服务 | 内置               |
| express.Router()     | 创建模块化路由   | 内置3.             |

#### 3.2 常用第三方中间件
| **中间件**      | **功能**          | **安装命令**                  |
| --------------- | ----------------- | ----------------------------- |
| morgan          | HTTP 请求日志记录 | `npm install morgan`          |
| helmet          | 安全头部设置      | `npm install helmet`          |
| cors            | 跨域资源共享支持  | `npm install cors`            |
| cookie-parser   | 解析 Cookie 数据  | `npm install cookie-parser`   |
| express-session | 会话管理          | `npm install express-session` |
| passport        | 身份认证          | `npm install passport`        |

#### 3.3 中间件配置示例
```javascript
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const morgan = require('morgan');

const app = express();

// 安全防护
app.use(helmet());

// 跨域支持
app.use(cors({
  origin: 'https://example.com',
  methods: ['GET', 'POST']
}));

// 请求日志
app.use(morgan('combined'));

// 静态文件服务
app.use('/public', express.static('public'));

// JSON 解析
app.use(express.json());
```

### 4. 项目结构最佳实践

#### 4.1 模块化项目结构
```
my-express-app/
├── config/               // 配置文件
│   └── database.js
├── controllers/          // 控制器
│   ├── userController.js
│   └── productController.js
├── routes/               // 路由定义
│   ├── api.js
│   └── web.js
├── models/               // 数据模型
│   └── User.js
├── middleware/           // 自定义中间件
│   └── auth.js
├── public/               // 静态文件
│   ├── css/
│   └── js/
├── views/                // 模板文件
│   └── index.ejs
├── app.js                // 主入口文件
└── package.json
```

#### 4.2 控制器示例
```javascript
// controllers/userController.js
exports.getAllUsers = async (req, res) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};

exports.createUser = async (req, res) => {
  try {
    const newUser = new User(req.body);
    await newUser.save();
    res.status(201).json(newUser);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
};
```

#### 4.3 路由整合
```javascript
// routes/api.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.get('/users', userController.getAllUsers);
router.post('/users', userController.createUser);

module.exports = router;

// app.js
const apiRouter = require('./routes/api');
app.use('/api', apiRouter);
```

### 5. 模板引擎集成

#### 5.1 支持的主流模板引擎
| **引擎**   | 语法特点          | 安装命令                         |
| ---------- | ----------------- | -------------------------------- |
| EJS        | 嵌入式 JavaScript | `npm install ejs`                |
| Pug        | 缩进式语法        | `npm install pug`                |
| Handlebars | Mustache 语法扩展 | `npm install express-handlebars` |
| Nunjucks   | Jinja2 风格       | `npm install nunjucks`           |

#### 5.2 EJS 集成示例
```javascript
// 设置模板引擎
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// 渲染页面
app.get('/profile', (req, res) => {
  res.render('profile', {
    user: {
      name: 'John Doe',
      email: 'john@example.com'
    }
  });
});

// views/profile.ejs
<html>
<body>
  <h1><%= user.name %></h1>
  <p>Email: <%= user.email %></p>
</body>
</html>
```

### 6. 高级开发技巧

#### 6.1 错误处理优化
```javascript
// 异步错误处理
const asyncHandler = fn => (req, res, next) => {
  Promise.resolve(fn(req, res, next))
    .catch(next);
};

app.get('/user/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) throw new Error('User not found');
  res.json(user);
}));

// 统一错误处理
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({
    error: {
      message: err.message,
      status: statusCode
    }
  });
});
```

#### 6.2 性能优化策略
```javascript
// 启用压缩
const compression = require('compression');
app.use(compression());

// 使用集群模式
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  app.listen(3000);
}

// 响应缓存
const apicache = require('apicache');
const cache = apicache.middleware;
app.get('/api/products', cache('5 minutes'), (req, res) => {
  // 返回产品数据
});
```

#### 6.3 安全加固实践
```javascript
// 安全头部
app.use(helmet());

// 防止跨站请求伪造 (CSRF)
const csrf = require('csurf');
app.use(csrf({ cookie: true }));

// 速率限制
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15分钟
  max: 100 // 每个IP限制100个请求
});
app.use(limiter);

// 输入验证
const { body, validationResult } = require('express-validator');
app.post('/register', 
  body('email').isEmail(),
  body('password').isLength({ min: 6 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // 处理注册
  }
);
```

### 7. Express 生态全景

#### 7.1 常用扩展框架
| **框架** | **基于** | **特点**                     |
| -------- | -------- | ---------------------------- |
| NestJS   | Express  | 面向企业级的 TypeScript 框架 |
| Koa      | -        | 更轻量、更现代的中间件架构   |
| Fastify  | -        | 高性能低开销框架             |
| Sails.js | Express  | MVC 框架，类似 Rails         |
| LoopBack | Express  | API 优先的框架               |

#### 7.2 开发工具链
| **工具**          | **用途**         |
| ----------------- | ---------------- |
| Express Generator | 快速搭建项目骨架 |
| Nodemon           | 开发热重载       |
| Postman           | API 测试         |
| Swagger           | API 文档生成     |
| PM2               | 生产环境进程管理 |

### 8. Express 应用部署指南

#### 8.1 部署准备
```bash
# 安装生产依赖
npm install --production

# 构建前端资源
npm run build

# 环境变量配置
cp .env.example .env
```

#### 8.2 PM2 进程管理
```bash
# 全局安装 PM2
npm install -g pm2

# 启动应用
pm2 start app.js --name "my-express-app"

# 常用命令
pm2 list         # 查看进程列表
pm2 logs         # 查看日志
pm2 monit        # 监控面板
pm2 save         # 保存当前配置
pm2 startup      # 设置开机启动
```

#### 8.3 Nginx 反向代理配置
```nginx
server {
  listen 80;
  server_name example.com;
  
  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
  
  # 静态文件直接由 Nginx 处理
  location /public/ {
    alias /path/to/app/public/;
    expires 1y;
  }
}
```

### 9. Express 5.0 新特性前瞻

#### 9.1 异步路由处理
```javascript
// 支持 async/await 无需额外包装
app.get('/user/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user);
});
```

#### 9.2 改进的错误处理
```javascript
// 自动捕获异步错误
app.get('/error', async (req, res, next) => {
  throw new Error('Something broke!');
  // 无需手动调用 next(err)
});
```

#### 9.3 增强的路由系统
```javascript
// 路由数组支持
const auth = require('./middleware/auth');
const admin = require('./middleware/admin');

app.get('/admin', [auth, admin], (req, res) => {
  res.send('Admin Area');
});
```

#### 9.4 内置 Promise 支持
```javascript
// 中间件可返回 Promise
app.use(async (req, res, next) => {
  await someAsyncOperation();
  next();
});
```

### 10. Express 最佳实践总结

1. **项目结构**：采用模块化组织（MVC 或分层架构）
2. **错误处理**：统一错误处理中间件 + 异步错误捕获
3. **安全防护**：Helmet + CORS + 速率限制 + 输入验证
4. **性能优化**：
   - 反向代理静态文件
   - 启用 Gzip 压缩
   - 使用缓存中间件
5. **开发效率**：
   - Nodemon 实现热重载
   - 使用 dotenv 管理环境变量
   - 编写集成测试（Jest + Supertest）

```javascript
// 测试示例
const request = require('supertest');
const app = require('../app');

describe('GET /users', () => {
  it('responds with JSON array', async () => {
    const response = await request(app)
      .get('/users')
      .expect('Content-Type', /json/)
      .expect(200);
    
    expect(response.body).toBeInstanceOf(Array);
  });
});
```

Express 以其简洁性、灵活性和强大的中间件生态，成为 Node.js Web 开发的首选框架。无论是构建简单的 REST API 还是复杂的企业级应用，Express 都能提供高效、可靠的解决方案。随着 Express 5.0 的发布，它将进一步融入现代 JavaScript 特性，保持其在 Node.js 生态中的核心地位。



## 十三、Express 中的 Request 和 Response 

Express 框架的核心在于它对 Node.js 原生 HTTP 模块的封装，特别是对请求（Request）和响应（Response）对象的增强。这些对象提供了丰富的属性和方法，极大简化了 Web 开发。

### 1. Request 对象（req）

Request 对象表示 HTTP 请求，包含客户端发送的所有信息。以下是其核心属性和方法：

### 1.1 基础属性
| **属性**          | **描述**                     | **示例**                          |
| ----------------- | ---------------------------- | --------------------------------- |
| `req.method`      | HTTP 请求方法                | `GET`, `POST`, `PUT`, `DELETE`    |
| `req.protocol`    | 请求协议 (`http` 或 `https`) | `'https'`                         |
| `req.secure`      | 是否通过 HTTPS 发起          | `true` (当 protocol 为 https)     |
| `req.hostname`    | 主机名 (不含端口号)          | `'example.com'`                   |
| `req.path`        | URL 路径部分                 | `'/users/profile'`                |
| `req.originalUrl` | 完整原始 URL                 | `'/users?id=123'`                 |
| `req.ip`          | 客户端 IP 地址               | `'192.168.1.100'`                 |
| `req.subdomains`  | 子域名数组                   | `['api', 'v2']`                   |
| `req.xhr`         | 是否是 AJAX 请求             | `true` (当 X-Requested-With 存在) |

### 1.2 参数获取
| **属性/方法**       | **描述**                           | **示例**                     |
| ------------------- | ---------------------------------- | ---------------------------- |
| `req.params`        | 路由参数对象                       | `{ userId: '123' }`          |
| `req.query`         | 查询字符串对象                     | `{ page: '2', limit: '10' }` |
| `req.body`          | 请求体数据 (需 body-parser 中间件) | `{ username: 'john' }`       |
| `req.cookies`       | Cookies 对象 (需 cookie-parser)    | `{ sessionId: 'abc123' }`    |
| `req.signedCookies` | 签名 Cookies 对象                  | `{ userId: '123' }`          |
| `req.get(field)`    | 获取请求头字段                     | `req.get('Content-Type')`    |

### 1.3 文件上传
```javascript
// 使用 multer 中间件
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('avatar'), (req, res) => {
  console.log(req.file); // 上传文件信息
  /*
  {
    fieldname: 'avatar',
    originalname: 'profile.jpg',
    encoding: '7bit',
    mimetype: 'image/jpeg',
    size: 12345,
    destination: 'uploads/',
    filename: 'abc123.jpg',
    path: 'uploads/abc123.jpg'
  }
  */
});
```

### 1.4 实用方法
```javascript
// 内容协商
const acceptType = req.accepts(['json', 'html']); 
// 返回客户端优先接受的类型

// 检查内容类型
if (req.is('json')) {
  // 处理 JSON 数据
}

// 范围请求
const range = req.range(1000); // 解析 Range 头
// [ { start: 0, end: 499 }, { start: 500, end: 999 } ]
```

### 2. Response 对象（res）

Response 对象用于构建和发送 HTTP 响应，提供多种发送响应的方法。

#### 2.1 响应发送方法
| **方法**               | **描述**                 | **示例**                      |
| ---------------------- | ------------------------ | ----------------------------- |
| `res.send(body)`       | 发送各种类型响应         | `res.send('Hello')`           |
| `res.json(obj)`        | 发送 JSON 响应           | `res.json({ success: true })` |
| `res.jsonp(obj)`       | 发送 JSONP 响应          | `res.jsonp({ data: [...] })`  |
| `res.sendFile(path)`   | 发送文件                 | `res.sendFile('/resume.pdf')` |
| `res.sendStatus(code)` | 发送状态码和对应状态消息 | `res.sendStatus(404)`         |
| `res.end()`            | 结束响应过程             | `res.end()`                   |

#### 2.2 响应头操作
| **方法**                   | **描述**          | **示例**                               |
| -------------------------- | ----------------- | -------------------------------------- |
| `res.status(code)`         | 设置状态码        | `res.status(201).send(...)`            |
| `res.set(field, value)`    | 设置响应头        | `res.set('Content-Type', 'text/html')` |
| `res.append(field, value)` | 追加响应头值      | `res.append('Set-Cookie', '...')`      |
| `res.type(type)`           | 设置 Content-Type | `res.type('png')`                      |
| `res.location(path)`       | 设置 Location 头  | `res.location('/new-path')`            |
| `res.vary(field)`          | 设置 Vary 头      | `res.vary('User-Agent')`               |

#### 2.3 重定向与视图渲染
```javascript
// 重定向
res.redirect('/new-location'); // 302 临时重定向
res.redirect(301, '/permanent-location'); // 301 永久重定向

// 视图渲染 (需模板引擎)
res.render('profile', { 
  user: req.user,
  title: '用户资料'
});

// 附件下载
res.download('/reports/sales.pdf'); // 自动设置 Content-Disposition
res.download('/data.csv', 'quarterly-report.csv'); // 自定义文件名
```

#### 2.4 Cookie 操作
```javascript
// 设置 Cookie
res.cookie('session', 'abc123', {
  maxAge: 24 * 60 * 60 * 1000, // 1天
  httpOnly: true,
  secure: true,
  sameSite: 'strict'
});

// 清除 Cookie
res.clearCookie('session', {
  httpOnly: true,
  secure: true
});
```

#### 2.5 高级响应方法
```javascript
// 流式响应
const fs = require('fs');
const stream = fs.createReadStream('large-video.mp4');
res.type('mp4');
stream.pipe(res);

// 分块传输
res.write('First chunk\n');
res.write('Second chunk\n');
res.end('Final chunk');

// 设置缓存
res.set('Cache-Control', 'public, max-age=3600'); // 1小时缓存
res.etag('md5-hash-value'); // 设置 ETag
res.lastModified(new Date()); // 设置最后修改时间
```

### 3. Request 和 Response 实际应用场景

#### 3.1 RESTful API 实现
```javascript
// 获取用户列表
app.get('/api/users', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  
  const users = UserService.getUsers(page, limit);
  res.json({
    data: users,
    page,
    limit,
    total: UserService.getTotalCount()
  });
});

// 创建用户
app.post('/api/users', (req, res) => {
  const newUser = req.body;
  
  if (!newUser.name || !newUser.email) {
    return res.status(400).json({ 
      error: '缺少必要字段' 
    });
  }
  
  const createdUser = UserService.createUser(newUser);
  res.status(201).location(`/api/users/${createdUser.id}`).json(createdUser);
});

// 更新用户
app.put('/api/users/:id', (req, res) => {
  const userId = req.params.id;
  const updates = req.body;
  
  const updatedUser = UserService.updateUser(userId, updates);
  if (!updatedUser) {
    return res.status(404).json({ error: '用户不存在' });
  }
  
  res.json(updatedUser);
});
```

#### 3.2 表单处理与文件上传
```javascript
const upload = multer({ 
  dest: 'uploads/',
  limits: { fileSize: 5 * 1024 * 1024 } // 5MB
});

app.post('/profile', upload.single('avatar'), (req, res) => {
  const formData = req.body;
  const avatarFile = req.file;
  
  if (!avatarFile) {
    return res.status(400).send('请上传头像');
  }
  
  // 处理表单数据
  UserService.updateProfile(req.user.id, {
    ...formData,
    avatarPath: avatarFile.path
  });
  
  res.redirect('/profile?success=true');
});
```

#### 3.3 内容协商
```javascript
app.get('/article/:id', (req, res) => {
  const article = ArticleService.getArticle(req.params.id);
  
  // 根据 Accept 头返回不同格式
  res.format({
    'text/html': () => {
      res.render('article', { article });
    },
    'application/json': () => {
      res.json(article);
    },
    'text/plain': () => {
      res.send(`标题: ${article.title}\n\n${article.content}`);
    },
    default: () => {
      res.status(406).send('不支持的格式');
    }
  });
});
```

#### 3.4 错误处理中间件
```javascript
// 自定义错误处理
app.use((err, req, res, next) => {
  // 记录错误日志
  console.error(`[${new Date().toISOString()}] ${req.method} ${req.url}`, err);
  
  // 设置响应状态码
  const status = err.status || 500;
  
  // 根据请求类型返回不同响应
  if (req.xhr) {
    res.status(status).json({
      error: {
        message: err.message || '服务器错误',
        code: err.code
      }
    });
  } else {
    res.status(status).render('error', {
      status,
      message: err.message || '服务器错误',
      stack: process.env.NODE_ENV === 'development' ? err.stack : null
    });
  }
});
```

### 4. Express 4.x 和 5.x 的差异

#### 4.1 Request 对象增强
| **特性**         | Express 4.x             | Express 5.x          |
| ---------------- | ----------------------- | -------------------- |
| 异步错误处理     | 需手动调用 next(err)    | 自动捕获 async 错误  |
| 参数解析         | 依赖 body-parser 中间件 | 内置更强大的解析器   |
| 路由参数类型转换 | 需手动转换              | 支持参数类型自动转换 |

#### 4.2 Response 对象改进
| **特性**         | Express 4.x    | Express 5.x              |
| ---------------- | -------------- | ------------------------ |
| res.send() 方法  | 不支持 Promise | 支持返回 Promise         |
| 流式响应错误处理 | 需手动监听错误 | 自动处理流错误           |
| 内容协商         | 有限支持       | 增强的 res.format() 方法 |

### 5. 最佳实践与性能技巧

#### 5.1 请求处理优化
```javascript
// 使用中间件过滤无效请求
app.use((req, res, next) => {
  // 检查无效的 User-Agent
  if (req.get('User-Agent').includes('BadBot')) {
    return res.status(403).send('Forbidden');
  }
  
  // 限制大文件上传
  if (req.get('Content-Length') > 10 * 1024 * 1024) {
    return res.status(413).send('文件过大');
  }
  
  next();
});

// 提前结束无效请求
app.get('/api/data', (req, res) => {
  if (!isValidToken(req.query.token)) {
    return res.sendStatus(401); // 提前返回
  }
  
  // 处理有效请求
  // ...
});
```

#### 5.2 响应优化策略
```javascript
// 启用压缩
const compression = require('compression');
app.use(compression());

// 设置缓存头
app.use('/static', express.static('public', {
  maxAge: '1d',
  setHeaders: (res, path) => {
    if (path.endsWith('.js')) {
      res.set('Content-Encoding', 'gzip');
    }
  }
}));

// 流式响应大文件
app.get('/large-file', (req, res) => {
  const fileStream = fs.createReadStream('large-data.bin');
  res.type('application/octet-stream');
  
  // 处理中断连接
  req.on('close', () => {
    fileStream.destroy();
  });
  
  fileStream.pipe(res);
});
```

### 6. 常见问题解决方案

#### 6.1 请求体无法解析
**问题**：`req.body` 始终为 undefined  
**解决**：
```javascript
// 添加合适的 body-parser 中间件
app.use(express.json()); // 解析 JSON
app.use(express.urlencoded({ extended: true })); // 解析表单
```

#### 6.2 文件上传问题
**问题**：文件上传失败或无法访问  
**解决**：
```javascript
// 确保正确配置 multer
const upload = multer({ 
  dest: 'uploads/',
  limits: { 
    fileSize: 10 * 1024 * 1024 // 10MB
  }
});

// 确保目录权限正确
fs.mkdirSync('uploads', { recursive: true });
```

#### 6.3 跨域请求问题
**解决**：
```javascript
// 使用 cors 中间件
const cors = require('cors');
app.use(cors({
  origin: 'https://example.com',
  methods: ['GET', 'POST'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

#### 6.4 Cookie 无法设置
**解决**：
```javascript
// 确保使用 cookie-parser
const cookieParser = require('cookie-parser');
app.use(cookieParser());

// 检查安全设置
res.cookie('session', 'value', {
  secure: true, // HTTPS 必需
  httpOnly: true,
  sameSite: 'Lax' // 或 'Strict'
});
```

### 7. 总结：核心要点回顾

#### 7.1 Request 对象关键点：
1. **参数获取**：
   - `req.params`：路由参数
   - `req.query`：查询字符串
   - `req.body`：请求体数据
2. **请求分析**：
   - `req.get()`：获取请求头
   - `req.accepts()`：内容协商
   - `req.is()`：检查内容类型
3. **文件处理**：
   - `req.file`：单个上传文件
   - `req.files`：多个上传文件

#### 7.2 Response 对象关键点：
1. **响应发送**：
   - `res.send()`：通用响应
   - `res.json()`：JSON 响应
   - `res.render()`：模板渲染
2. **流控制**：
   - `res.status()`：设置状态码
   - `res.set()`：设置响应头
   - `res.redirect()`：重定向
3. **高级功能**：
   - `res.download()`：文件下载
   - `res.format()`：内容协商
   - `res.cookie()`：Cookie 管理

#### 7.3 最佳实践：
1. **请求处理**：
   - 尽早验证输入参数
   - 使用中间件处理公共逻辑
   - 合理限制请求大小
2. **响应优化**：
   - 设置缓存头减少重复请求
   - 使用流处理大文件
   - 启用压缩减少传输大小
3. **错误处理**：
   - 统一错误响应格式
   - 区分客户端和服务端错误
   - 记录详细错误日志

Express 的请求和响应对象是 Web 开发的核心工具，通过熟练掌握它们的属性和方法，可以高效构建健壮的 Web 应用和 API 服务。



## 十四、EJS 模板引擎

EJS (Embedded JavaScript) 是 Node.js 生态中最流行的模板引擎之一，它允许开发者在 HTML 中嵌入 JavaScript 代码，实现动态内容渲染。EJS 以其简洁的语法和强大的功能成为构建动态 Web 应用的首选工具。

### 1. EJS 核心特性

#### 1.1 核心优势

- **简单易学**：语法类似 HTML，学习曲线平缓
- **JavaScript 嵌入**：直接在模板中使用 JS 逻辑
- **组件复用**：支持模板片段和布局
- **快速高效**：编译为 JavaScript 函数，执行速度快
- **灵活扩展**：支持自定义过滤器和标签

#### 1.2 语法特点
| **语法** | **作用**             | **示例**                         |
| -------- | -------------------- | -------------------------------- |
| `<%`     | 执行 JavaScript 代码 | `<% for(let i=0; i<5; i++) { %>` |
| `<%=`    | 输出转义后的值       | `<%= user.name %>`               |
| `<%-`    | 输出原始 HTML        | `<%- '<strong>Bold</strong>' %>` |
| `%>`     | 结束标签             | `<% } %>`                        |
| `<%#`    | 注释                 | `<%# This is a comment %>`       |

### 2. EJS 安装与基础使用

#### 2.1 安装
```bash
npm install ejs
```

#### 2.2 Express 集成
```javascript
const express = require('express');
const app = express();

// 设置 EJS 为模板引擎
app.set('view engine', 'ejs');

// 设置模板目录 (默认为 views)
app.set('views', path.join(__dirname, 'templates'));

// 渲染模板
app.get('/', (req, res) => {
  res.render('index', {
    title: 'EJS 首页',
    user: { name: '张三', age: 28 },
    items: ['苹果', '香蕉', '橙子']
  });
});
```

#### 2.3 基础模板示例
```ejs
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1>欢迎, <%= user.name %>!</h1>
  
  <% if (user.age >= 18) { %>
    <p>您已成年</p>
  <% } else { %>
    <p>您未成年</p>
  <% } %>
  
  <ul>
    <% items.forEach(item => { %>
      <li><%= item %></li>
    <% }) %>
  </ul>
  
  <%- include('footer') %>
</body>
</html>
```

### 3. EJS 高级功能详解

#### 3.1 布局系统 (Layouts)
```ejs
<!-- views/layout.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
  <%- include('partials/head') %>
</head>
<body>
  <%- include('partials/header') %>
  
  <main>
    <%- body %> <!-- 内容插入点 -->
  </main>
  
  <%- include('partials/footer') %>
</body>
</html>

<!-- views/home.ejs -->
<% layout('layout') -%>

<h1>欢迎来到首页</h1>
<p>当前时间: <%= new Date().toLocaleString() %></p>
```

#### 3.2 包含部分视图 (Partials)
```ejs
<!-- views/partials/header.ejs -->
<header>
  <nav>
    <a href="/">首页</a>
    <a href="/about">关于</a>
  </nav>
</header>

<!-- 在模板中使用 -->
<%- include('partials/header') %>
```

#### 3.3 自定义分隔符
```javascript
// 在 Express 中配置
const ejs = require('ejs');
ejs.delimiter = '?'; // 使用 ? 替代 %

// 使用新分隔符
app.engine('ejs', ejs.renderFile);
```

#### 3.4 过滤器功能
```javascript
// 注册自定义过滤器
ejs.filters.currency = function(value) {
  return '¥' + parseFloat(value).toFixed(2);
};

// 在模板中使用
<p>价格: <%=: price | currency %></p>
```

### 4. EJS 最佳实践

#### 4.1 模板组织策略
```
views/
├── layouts/
│   └── main.ejs
├── partials/
│   ├── header.ejs
│   ├── footer.ejs
│   └── head.ejs
├── pages/
│   ├── home.ejs
│   ├── about.ejs
│   └── contact.ejs
└── components/
    ├── product-card.ejs
    └── user-profile.ejs
```

#### 4.2 数据传递与处理
```javascript
// 控制器中传递数据
res.render('product', {
  product: {
    id: 123,
    name: '智能手机',
    price: 2999.99,
    features: ['大屏幕', '长续航', '高清相机']
  },
  helpers: {
    formatDate: (date) => date.toLocaleDateString()
  }
});

// 模板中使用
<h2><%= product.name %></h2>
<p>发布日期: <%= helpers.formatDate(new Date()) %></p>
```

#### 4.3 避免过度逻辑
```ejs
<!-- 不推荐: 复杂逻辑在模板中 -->
<% 
  let discount = 0;
  if (user.vipLevel > 3) {
    discount = 0.2;
  } else if (user.vipLevel > 1) {
    discount = 0.1;
  }
%>

<!-- 推荐: 逻辑在控制器中处理 -->
<!-- 控制器 -->
res.render('product', {
  discount: calculateDiscount(user.vipLevel)
});

<!-- 模板中 -->
<p>折扣: <%= discount * 100 %>%</p>
```

#### 4.4 XSS 防护策略
```ejs
<!-- 安全做法: 输出转义 -->
<p><%= userInput %></p> <!-- 自动转义 HTML -->

<!-- 需要原始 HTML 时明确标识 -->
<p><%- sanitizedHtml %></p> <!-- 确保内容已消毒 -->

<!-- 控制器中对用户输入消毒 -->
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');
const window = new JSDOM('').window;
const DOMPurify = createDOMPurify(window);

const sanitizedHtml = DOMPurify.sanitize(userInput);
```

### 5. EJS 性能优化

#### 5.1 模板缓存
```javascript
// 开发环境禁用缓存
app.set('view cache', false);

// 生产环境启用缓存
if (process.env.NODE_ENV === 'production') {
  app.set('view cache', true);
}
```

#### 5.2 预编译模板
```javascript
const ejs = require('ejs');
const fs = require('fs');

// 预编译模板
const template = fs.readFileSync('views/template.ejs', 'utf8');
const compiledTemplate = ejs.compile(template);

// 渲染时使用编译后的函数
app.get('/page', (req, res) => {
  const html = compiledTemplate({ data: 'value' });
  res.send(html);
});
```

#### 5.3 异步渲染
```javascript
// 使用 ejs.renderFile 的 promise 版本
const util = require('util');
const renderFile = util.promisify(ejs.renderFile);

app.get('/async-page', async (req, res) => {
  try {
    const html = await renderFile('views/async.ejs', {
      fetchData: await fetchSomeData()
    });
    res.send(html);
  } catch (err) {
    res.status(500).send('渲染错误');
  }
});
```

### 6. 与其他模板引擎对比

| **特性**     | EJS           | Pug      | Handlebars    | Nunjucks    |
| ------------ | ------------- | -------- | ------------- | ----------- |
| **语法**     | HTML 嵌入 JS  | 缩进式   | Mustache 风格 | Jinja2 风格 |
| **学习曲线** | 低            | 中       | 低            | 中          |
| **性能**     | 高            | 中       | 中            | 中          |
| **逻辑支持** | 完整 JS       | 完整 JS  | 有限          | 完整 JS     |
| **布局系统** | 需要扩展      | 内置     | 需要扩展      | 内置        |
| **部分视图** | 支持          | 支持     | 支持          | 支持        |
| **适合场景** | 传统 Web 应用 | 简洁模板 | 简单模板      | 复杂模板    |

### 7. EJS 常见问题解决方案

#### 7.1 变量未定义错误
```ejs
<!-- 安全访问嵌套属性 -->
<p><%= user && user.address && user.address.city %></p>

<!-- 或使用可选链 (Node.js 14+) -->
<p><%= user?.address?.city %></p>
```

#### 7.2 包含文件路径问题
```ejs
<!-- 使用绝对路径 -->
<%- include('/partials/header') %>

<!-- 或设置根目录 -->
const ejs = require('ejs');
ejs.root = path.join(__dirname, 'views');
```

#### 7.3 处理换行问题
```ejs
<!-- 使用 '-' 移除换行 -->
<% if (user) { -%>
  <p>欢迎回来</p>
<% } -%>
```

#### 7.4 国际化支持
```javascript
// 注册国际化过滤器
ejs.filters.t = function(key) {
  return i18n.t(key);
};

// 模板中使用
<h1><%=: 'welcome_message' | t %></h1>
```

### 8. EJS 实战案例

#### 8.1 动态表格生成
```ejs
<table>
  <thead>
    <tr>
      <% Object.keys(products[0]).forEach(key => { %>
        <th><%= key %></th>
      <% }) %>
    </tr>
  </thead>
  <tbody>
    <% products.forEach(product => { %>
      <tr>
        <% Object.values(product).forEach(value => { %>
          <td><%= value %></td>
        <% }) %>
      </tr>
    <% }) %>
  </tbody>
</table>
```

#### 8.2 分页组件
```ejs
<div class="pagination">
  <% if (currentPage > 1) { %>
    <a href="/items?page=<%= currentPage-1 %>">上一页</a>
  <% } %>
  
  <% for(let i=1; i<=totalPages; i++) { %>
    <% if (i === currentPage) { %>
      <span class="current"><%= i %></span>
    <% } else { %>
      <a href="/items?page=<%= i %>"><%= i %></a>
    <% } %>
  <% } %>
  
  <% if (currentPage < totalPages) { %>
    <a href="/items?page=<%= currentPage+1 %>">下一页</a>
  <% } %>
</div>
```

#### 8.3 条件样式
```ejs
<div class="alert alert-<%= message.type %>">
  <%= message.text %>
</div>

<style>
  .alert-success { color: green; }
  .alert-error { color: red; }
  .alert-warning { color: orange; }
</style>
```

### 9. EJS 扩展与进阶

#### 9.1 客户端使用 EJS
```html
<!-- 浏览器中直接使用 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/ejs/3.1.8/ejs.min.js"></script>

<script>
  const template = `
    <h1><%= title %></h1>
    <ul>
      <% items.forEach(item => { %>
        <li><%= item %></li>
      <% }) %>
    </ul>
  `;
  
  const html = ejs.render(template, {
    title: "客户端渲染",
    items: ["项目1", "项目2"]
  });
  
  document.getElementById('app').innerHTML = html;
</script>
```

#### 9.2 与前端框架集成
```javascript
// 在 React 中使用 EJS 模板
import ejs from 'ejs';
import template from './template.ejs';

function EjsComponent({ data }) {
  const html = ejs.render(template, data);
  
  return (
    <div dangerouslySetInnerHTML={{ __html: html }} />
  );
}
```

#### 9.3 自定义标签
```javascript
// 创建自定义标签
ejs.open = '{{';
ejs.close = '}}';

// 模板中使用新标签
{{= user.name }}
```

### 10、总结：EJS 核心价值

1. **简单直观**：  
   - 基于 HTML 的模板语法
   - 直接嵌入 JavaScript 逻辑
   - 无需学习新语法体系

2. **灵活强大**：  
   - 支持布局和部分视图
   - 可扩展的过滤器和标签
   - 完整的 JavaScript 能力

3. **高性能**：  
   - 模板编译缓存
   - 高效的渲染机制
   - 支持流式输出

4. **全栈适用**：  
   - 服务端渲染 (Node.js)
   - 客户端渲染 (浏览器)
   - 静态网站生成

5. **生态丰富**：  
   - 与 Express 完美集成
   - 支持多种模板组织模式
   - 丰富的扩展插件

```ejs
<%# EJS 是 Node.js 开发者的首选模板引擎之一 %>
<%- include('partials/cta', { 
  text: '开始使用 EJS', 
  url: 'https://ejs.co' 
}) %>
```

通过掌握 EJS，您可以高效地构建动态 Web 页面，分离业务逻辑与展示层，创建可维护的模板系统。无论是简单的博客还是复杂的企业应用，EJS 都能提供灵活、高效的解决方案。