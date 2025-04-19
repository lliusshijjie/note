# Redis(Remote Dictionary Server) 数据库



## 一. NoSQL数据库介绍



NoSQL（Not Only SQL）是一类非关系型数据库管理系统，用于存储和检索非关系型数据。与传统的关系型数据库(RDBMS)不同，NoSQL数据库不使用SQL作为查询语言，通常也不遵循严格的表结构。

#### NoSQL数据库的主要特点

1. **灵活的数据模型**：不需要预定义严格的表结构
2. **水平可扩展性**：易于分布式部署和扩展
3. **高性能**：针对特定类型的查询和数据操作进行了优化
4. **高可用性**：通常内置复制和容错机制
5. 

#### 主要NoSQL数据库类型

1. __键值存储(Key-Value)__

- **特点**：最简单的NoSQL形式，数据作为键值对存储
- **代表产品**：Redis, DynamoDB, Riak
- **适用场景**：会话存储、缓存、配置数据

2. __文档型数据库__

- **特点**：存储半结构化文档(如JSON, BSON, XML)
- **代表产品**：MongoDB, CouchDB
- **适用场景**：内容管理系统、用户配置、产品目录

3. __列族存储__

- **特点**：按列而不是按行存储数据
- **代表产品**：Cassandra, HBase
- **适用场景**：大数据分析、日志系统

4. __图数据库__

- **特点**：专注于实体之间的关系
- **代表产品**：Neo4j, ArangoDB
- **适用场景**：社交网络、推荐系统、欺诈检测



#### NoSQL的优势

- 处理大规模数据
- 高吞吐量
- 灵活的数据模型
- 易于水平扩展
- 适合非结构化或半结构化数据



#### NoSQL的局限性

- 缺乏标准化
- 事务支持有限(某些类型)
- 查询能力可能不如SQL丰富
- 学习曲线因产品而异

NoSQL不是关系型数据库的替代品，而是为特定用例设计的补充解决方案。选择数据库类型应根据具体应用需求和数据特性来决定。



__Redis:__远程词典服务器，是一个基于内存的键值型的NoSQL数据库

__特征：__

1. 键值（key-value）型，value支持多种不同数据结构，功能丰富
2. 单线程，每一个命令具有原子性
3. 低延迟，速度快（__基于内存__，IO多路复用，良好的编码）
4. 支持数据持久化
5. 支持主从集群，分片集群
6. 支持多语言客户端



## 二. Redis 数据结构详解

Redis 是一个高性能的键值存储系统，它支持多种数据结构，而不仅仅是简单的字符串键值对。以下是 Redis 主要支持的数据结构及其特性：

## 1. 字符串 (String)
- **特点**：最基本的数据类型，二进制安全的，可以包含任何数据

- **最大容量**：512MB

- **常用命令**：
  - `SET key value` - 设置键值
  - `GET key` - 获取值
  - `INCR key` - 值递增
  - `APPEND key value` - 追加值
  
- **应用场景**：缓存、计数器、分布式锁

  

## 2. 哈希 (Hash)
- **特点**：键值对集合，适合存储对象

- **最大容量**：每个哈希可存储 2³² - 1 键值对

- **常用命令**：
  - `HSET key field value` - 设置字段值
  - `HGET key field` - 获取字段值
  - `HGETALL key` - 获取所有字段值
  - `HDEL key field` - 删除字段
  
- **应用场景**：存储用户信息、商品属性等对象数据

  

## 3. 列表 (List)
- **特点**：有序的字符串列表，按照插入顺序排序

- **最大容量**：2³² - 1 个元素

- **常用命令**：
  - `LPUSH key value` - 左侧插入
  - `RPUSH key value` - 右侧插入
  - `LPOP key` - 左侧弹出
  - `LRANGE key start stop` - 获取范围元素
  
- **应用场景**：消息队列、最新消息排行、记录日志

  

## 4. 集合 (Set)
- **特点**：无序的字符串集合，不允许重复成员

- **最大容量**：2³² - 1 个成员

- **常用命令**：
  - `SADD key member` - 添加成员
  - `SMEMBERS key` - 获取所有成员
  - `SINTER key1 key2` - 交集
  - `SUNION key1 key2` - 并集
  
- **应用场景**：标签系统、共同好友、唯一性检查

  

## 5. 有序集合 (Sorted Set / ZSet)
- **特点**：与集合类似，但每个成员关联一个分数(score)，用于排序

- **最大容量**：2³² - 1 个成员

- **常用命令**：
  - `ZADD key score member` - 添加成员
  - `ZRANGE key start stop` - 按分数范围获取成员
  - `ZREVRANK key member` - 获取成员排名(降序)
  
- **应用场景**：排行榜、带权重的队列

  

## 6. 位图 (Bitmap)
- **特点**：通过字符串实现的位操作数据结构

- **常用命令**：
  - `SETBIT key offset value` - 设置位
  - `GETBIT key offset` - 获取位
  - `BITCOUNT key` - 统计设置为1的位数
  
- **应用场景**：用户签到、活跃用户统计

  

## 7. HyperLogLog
- **特点**：用于基数统计的算法，提供不精确的去重计数

- **常用命令**：
  - `PFADD key element` - 添加元素
  - `PFCOUNT key` - 估算基数
  - `PFMERGE destkey sourcekey` - 合并多个HyperLogLog
  
- **应用场景**：大规模数据去重统计(如UV统计)

  

## 8. 地理空间 (Geospatial)
- **特点**：存储地理位置信息

- **常用命令**：
  - `GEOADD key longitude latitude member` - 添加位置
  - `GEODIST key member1 member2` - 计算距离
  - `GEORADIUS key longitude latitude radius` - 查找半径内成员
  
- **应用场景**：附近的人、位置服务

  

## 9. 流 (Stream)
- **特点**：Redis 5.0引入，类似日志的数据结构，用于消息队列
- **常用命令**：
  - `XADD key ID field value` - 添加消息
  - `XREAD COUNT n STREAMS key ID` - 读取消息
  - `XGROUP CREATE key groupname ID` - 创建消费者组
- **应用场景**：消息队列、事件溯源

Redis 的丰富数据结构使其能够高效地解决各种不同场景下的数据存储和处理问题，这也是 Redis 强大功能的核心所在。



## 三. Redis 常用命令大全

## 键(Key)相关命令

1. **基本操作**
   - `SET key value` - 设置键值
   - `GET key` - 获取键值
   - `DEL key` - 删除键
   - `EXISTS key` - 检查键是否存在
   - `EXPIRE key seconds` - 设置键的过期时间(秒)
   - `TTL key` - 查看键剩余生存时间

2. **批量操作**
   - `MSET key1 value1 key2 value2` - 批量设置键值
   
   - `MGET key1 key2` - 批量获取键值
   
   - `KEYS pattern` - 查找匹配模式的键(生产环境慎用)
   
     

## 字符串(String)命令

1. **基本操作**
   
   - `APPEND key value` - 追加值
   - `STRLEN key` - 获取字符串长度
   - `SETNX` - 添加一个String类型的键值对，前提是这个key不存在，否则不执行
   - `SETEX` - 添加一个String类型的键值对，并且指定有效期
   
2. **数字操作**
   - `INCR key` - 值递增1
   
   - `INCRBY key increment` - 值增加指定数值
   
   - `INCRBYFLOAT` - 让一个浮点类型的数字自增并指定步长
   
   - `DECR key` - 值递减1
   
   - `DECRBY key decrement` - 值减少指定数值
   
     

## 哈希(Hash)命令

1. **字段操作**
   - `HSET key field value` - 设置哈希字段值
   - `HGET key field` - 获取哈希字段值
   - `HDEL key field` - 删除哈希字段
   - `HEXISTS key field` - 检查字段是否存在

2. **批量操作**
   - `HMSET key field1 value1 field2 value2` - 批量设置字段值
   - `HMGET key field1 field2` - 批量获取字段值
   - `HGETALL key` - 获取所有字段和值

3. **其他操作**
   - `HKEYS key` - 获取所有字段名
   
   - `HVALS key` - 获取所有字段值
   
   - `HLEN key` - 获取字段数量
   
     

## 列表(List)命令

1. **基本操作**
   - `LPUSH key value` - 从列表左侧插入
   - `RPUSH key value` - 从列表右侧插入
   - `LPOP key` - 从左侧弹出元素
   - `RPOP key` - 从右侧弹出元素

2. **查询操作**
   - `LRANGE key start stop` - 获取列表范围内的元素
   - `LLEN key` - 获取列表长度
   - `LINDEX key index` - 通过索引获取元素

3. **高级操作**
   - `BLPOP key timeout` - 阻塞式左侧弹出(队列)
   - `BRPOP key timeout` - 阻塞式右侧弹出(队列)



## 集合(Set)命令

1. **成员操作**
   - `SADD key member` - 添加成员
   - `SREM key member` - 删除成员
   - `SISMEMBER key member` - 检查成员是否存在

2. **集合操作**
   - `SMEMBERS key` - 获取所有成员
   - `SCARD key` - 获取成员数量
   - `SRANDMEMBER key [count]` - 随机获取成员

3. **集合运算**
   - `SINTER key1 key2` - 交集
   - `SUNION key1 key2` - 并集
   - `SDIFF key1 key2` - 差集



## 有序集合(Sorted Set)命令

1. **成员操作**
   - `ZADD key score member` - 添加成员
   - `ZREM key member` - 删除成员
   - `ZSCORE key member` - 获取成员分数

2. **查询操作**
   - `ZRANGE key start stop [WITHSCORES]` - 按分数升序获取成员
   - `ZREVRANGE key start stop [WITHSCORES]` - 按分数降序获取成员
   - `ZRANK key member` - 获取成员排名(升序)
   - `ZREVRANK key member` - 获取成员排名(降序)

3. **范围查询**
   - `ZRANGEBYSCORE key min max` - 按分数范围查询
   - `ZCOUNT key min max` - 统计分数范围内的成员数



## 发布订阅命令

1. **基本操作**
   - `PUBLISH channel message` - 发布消息到频道
   - `SUBSCRIBE channel` - 订阅频道
   - `UNSUBSCRIBE [channel]` - 取消订阅



## 事务命令

1. **基本操作**
   - `MULTI` - 开始事务
   - `EXEC` - 执行事务
   - `DISCARD` - 取消事务
   - `WATCH key` - 监视键(乐观锁)



## 服务器命令

1. **管理命令**
   - `INFO [section]` - 获取服务器信息
   - `CONFIG GET parameter` - 获取配置参数
   - `FLUSHDB` - 清空当前数据库
   - `FLUSHALL` - 清空所有数据库

2. **持久化**
   - `SAVE` - 同步保存数据到磁盘
   - `BGSAVE` - 异步保存数据到磁盘

这些是Redis最常用的核心命令，掌握它们可以满足大多数日常开发需求。