# NodeJS

菜鸟教程：https://www.runoob.com/nodejs/nodejs-tutorial.html

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

### 4.包管理与生态（npm）—— 扩展能力的引擎**

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