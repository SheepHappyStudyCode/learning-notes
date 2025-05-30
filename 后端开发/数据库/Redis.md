﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿# Redis
## 1 Redis入门知识
1. Redis简介：远程词典服务器，是一种**基于内存**的**键值型**NoSQL数据库。
2. NoSQL数据库：非关系型数据库。
	- 特点：
		- 非结构化
		- 无关联的
		- 非SQL
		- BASE（不满足ACID的事务特性）
		- 数据存放在内存中
		- 具有水平扩展性

	- 使用场景：
		- 数据结构不固定
		- 对一致性、安全性要求不高
		- 对性能要求高

## 2 常用数据结构

### String

最基本的数据类型，本质是一个字节数组，可以存放普通字符串，也可以存放数字。

### List

双向链表

### Hash

一个键值对集合

### Set

没有顺序的哈希表

### Zset

相比于 Set 多了一个 score 字段，常用于实现实时排行榜功能

### Bitmap

位数组，可以用于保存大量简单的状态，例如用户的连续登录天数，每天的用户登录数量

### HyperLogLog

可以用来估算大规模数据集的基数，它只关心不同元素的数量，而不是元素的具体值，需要耗费的内存一定，估算的偏差也在一定范围。

### Geospatial
记录和处理地理信息



## 3 Redis 的事务操作
1. 事务的作用：串联多个命令，防止其他命令插队。
2. 事务命令：
	- `Multi`：组队阶段，输入的命令都进入命令序列，但不会执行。
	- `Exec`：执行命令
	- `discard`：放弃组队
3. 错误处理：
	- 组队阶段出现错误：全部命令都不会执行
	- 执行阶段出现错误：错误命令不会执行，其他命令都执行



## 4 Redis持久化

### RDB 持久化

在指定的**时间间隔**内将内存中的数据集**快照**写入磁盘。

**优点：**

从 RDB 文件恢复数据的速度很快，且文件占用体积比 AOF 小。

**缺点：**

两次保存数据库快照的时间间隔较长，如果宕机容易发生数据丢失。

### AOF 持久化

以**日志**的形式记录每个写操作（增量保存）。

**优点：**

默认是每秒写入日志一次，**数据的安全性高**，不容易发生丢失

**缺点：**

从 AOF 文件恢复数据的速度比 RDB 慢，AOF 文件的大小通常也比 RDB 大。（不过 AOF 文件到达一定阈值会触发重写）

#### AOF 文件的重写

AOF 重写是一个后台操作，不会阻塞 Redis 服务，它会通过 **fork 创建一个子进程**，子进程会读取 Redis 的 **内存数据** 写到一个临时文件中，最后替换掉原来的文件。

（PS：AOF 文件重写期间的命令会被主节点写到缓冲区里，重写完成后主线程会将这些命令写入新的 AOF 文件。）

## 混合持久化

> RDB 和 AOF 各有优缺点，Redis 4.0 以后引入了混合持久化

具体来说，当触发 AOF 文件重写时，会先将此时的数据以 RDB 持久化的方式写在新 AOF 文件的头部，重写期间的写操作追加到 AOF 文件。

## 5 Redis的主从复制
### 5.1 简介
主从复制是指一个 Redis 实例（主节点）可以将数据复制到一个或多个从节点，实现数据备份。
### 5.2 作用
1. 数据备份
2. 容灾快速恢复（配合 Redis 哨兵模式能够完成故障**自动转移**）
3. 读写分离
### 5.3 相关命令
1. 查看主从信息：`info replication`
2. 配置主从信息：`slaveof <host> <port>` 
3. 从机替代主机：`slaveof no one`
### 5.4 哨兵模式 sentinel
1. 简介：常配合主从复制的 Redis 节点使用，可以实现**自动故障转移**。

2. 替换规则：
	- 要设置至少一个哨兵监视主机。
	- 优先级高的从机优先替换。
	- 从机替换主机后，原来的主机重启后会成为从机。
	
### 5.5 主从复制的原理

从节点向主节点发送同步请求，主节点根据情况使用**全量复制**或**增量复制**。

#### 全量复制

从节点第一次连接到主节点或者从节点与主节点的数据差异较大的时候使用

**流程：**

	1. 从节点向主节点发送同步请求
	1. 主节点接收到请求后使用 basave 命令生成 RDB 快照，发给从节点，期间的写命令会记录在一个缓冲区里
	1. 从节点接收到 RDB 快照文件进行全量复制，全量复制完成后主节点将缓冲区的数据发给从节点。

#### 增量复制

主节点会维护一个 `repl_backlog_buffer `，是一个环形缓冲区，记录最近的读写操作，从节点发送同步请求时，如果发现从节点的 offset 还在缓冲区里，就会使用增量复制。

## 6 Redis 分片集群

1. 简介：Redis集群将Redis进行**水平扩容**，即启动N个Redis节点，将整个数据库分布存储在这N个节点中。一般采用**无中心化**集群。
2. 集群化配置：
``` 
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
```
3. 相关命令：
	- 查看集群信息：`cluster nodes`
	- 查看键的插槽值：`cluster keyslot <key>`
	- 计算插槽值里有几个键：`cluster countkeysinslot <插槽值>`
	- 得到插槽中的键：`cluster getkeysinslot <插槽值>`
4. 集群分配原则：
	- 集群至少有**三个主节点**
	- 尽量保证每个主数据库运行在**不同的IP地址**，每个从库和主库不在同一个IP地址上。
5. 优点：
	- 实现扩容
	- 分摊压力
	- 无中心配置相对简单
6. 缺点：
	- 多键操作不被支持
	- 多建的Redis事务不被支持。lua脚本不支持
	- 很多公司的采用其他的集群方案，迁移困难
## 7 Redis应用问题解决
### 7.1 缓存穿透
1. 问题描述：很多请求**查询数据库不存在的数据**，缓存对这类请求没有效果，导致直接查询数据库，导致数据库压力激增。
2. 解决方法：
	1. **缓存一些空值**，设置较短的过期时间
	2. 设置白名单，屏蔽恶意访问的 IP。
	3. 采用**布隆过滤器**，提前判断一些数据存不存在，不存在直接返回。
### 7.2 缓存雪崩

1. 问题描述：大量 key 几乎同时失效，导致大量请求同时查询数据库。
2. 解决方案：
   - 如果是大量 key 同时失效导致的缓存雪崩，可以设置 key 的**过期时间随机化**
   - 如果是 Redis 宕机导致的缓存雪崩，可以构建集群保证 Redis 的高可用。

### 7.3 缓存击穿
1. 问题描述：一些**热点 key** 在较短时间内同时过期，导致大量并发请求查询数据库。
2. 解决方案：
	- **互斥锁**：只允许一个线程查询数据库，这个线程会进行缓存重建，其他线程就去查缓存。
	- **预先设置热门数据**，延长热门数据key的时长，甚至**永不过期**。
	- 热点 key 的过期时间随机化。
### 7.4 分布式锁
1. Redia实现分布式锁：通过`setnx`插入键值对，只有`del`这个键值对后才能再次赋值。同时要对这个键值对设置过期时间，避免锁长时间不释放。
2. 误删锁问题的解决方案：
	- 拿到锁时设置一个唯一标识，删除前判断该标识是否发生变化，如果是自己的标识才释放锁。（避免释放别人的锁）
	- 使用Lua脚本实现原子性操作
## 8 Redis新功能
### 11.1 ACL
1. ACL：权限控制列表
2. 相关命令：
	- `acl list`
	- `acl cat`
	- `acl setuser <username>`
	- `acl whoami`
### 11.2 IO多线程
1. 虽然支持多线程，但执行命令的方式还是单线程。

### 11.3 工具支持Cluster



## 9 Redis客户端的常见命令

1. [springboot使用redis（StringRedisTemplate的用法）-CSDN博客](https://blog.csdn.net/weixin_43835717/article/details/92802040)

2. [在 Spring Boot 中整合、使用 Redis - spring 中文网 (springdoc.cn)](https://springdoc.cn/spring-boot-data-redis/)

3. [redisson使用全解——redisson官方文档+注释（上篇）_redisson官网中文-CSDN博客](https://blog.csdn.net/A_art_xiang/article/details/125525864)

4. [【进阶篇】Redis实战之Redisson使用技巧详解，干活！-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2223087)



## 10 Redis 的底层知识

### 10.1 跳跃表（Skip List）

Redis 中的**跳跃表**是一种用于实现**有序集合（Sorted Set）**底层数据结构。它是一种**概率性**的数据结构，提供了高效的插入、删除和查找操作，平均时间复杂度为 **O(log N)**。

跳跃表的结构基于**多个层级的链表**，每一层链表都是上一层链表的一个子集。每一层的链表通过跳跃的方式来加速查找操作。

**跳跃表的优势**

1. 高效的查询效率：每一层通过跳跃加快查询效率，整个查询过程类似二分查找。
2. 平衡性：不像二叉树需要以自旋的方式维持平衡，跳跃表通过概率的提升来实现平衡。
3. 空间复杂度较低：比红黑树省空间，因为不需要存储类似父节点这样的信息。



### 10.2 Redis 线程模型

Redis 采用的是一种**单线程事件驱动模型**，使得可以用少量的线程处理多个 Socket 上的事件，大大提高并发能力。

#### 核心组件

1. IO 多路复用程序
2. 事件队列
3. 文件事件分发器
4. 三个事件处理器：连接应答处理器、命令请求处理器、命令回复处理器

#### 请求处理流程

1. 建立连接：IO 多路复用程序将连接建立事件放入事件队列，分发器将其交给连接应答处理器，连接应答处理器为每一个客户端建立对应的 Socket。
2. 处理请求：将读写命令分发给命令请求处理器，处理完命令后，将命令完成事件交给事件队列。
3. 事件分发器将命令完成事件交给命令回复处理器，命令回复处理器返回给客户端 OK，并将 Socket 解绑。

#### 讲解事件驱动模型的好文

[事件驱动架构在 vivo 内容平台的实践本文前半部分重点阐述事件驱动架构的定义和重要概念，以及架构设计的场景和原因分析， - 掘金](https://juejin.cn/post/7056577036688556062)



## Redisson 源码

### 1 RedissonRateLimiter 源码

`rateLimiter.trySetRate(rateType, rate, duration);`

```java
return this.commandExecutor.evalWriteNoRetryAsync(
    this.getRawName(),             // 限流器的名称
    LongCodec.INSTANCE,            // 使用的编解码器
    RedisCommands.EVAL_BOOLEAN,    // Redis命令类型(执行Lua脚本并返回布尔值)
    
    // Lua脚本内容
    "redis.call('hsetnx', KEYS[1], 'rate', ARGV[1]);" +                // 设置速率
    "redis.call('hsetnx', KEYS[1], 'interval', ARGV[2]);" +            // 设置时间间隔
    "local res = redis.call('hsetnx', KEYS[1], 'type', ARGV[3]);" +    // 设置限流类型
    "if res == 1 and tonumber(ARGV[4]) > 0 then " +                    // 如果是首次设置且过期时间>0
    "   redis.call('pexpire', KEYS[1], ARGV[4]); " +                   // 设置过期时间
    "end; " +
    "return res;",                                                     // 返回设置类型的结果
    
    // 参数列表
    Collections.singletonList(this.getRawName()),                      // KEYS数组
    new Object[]{rate, rateInterval.toMillis(), type.ordinal(), timeToLive.toMillis()} // ARGV数组
);
```

`rateLimiter.tryAcquire()`

```java
private <T> RFuture<T> tryAcquireAsync(RedisCommand<T> command, Long value) {
    // 生成随机ID，用于在Redis中唯一标识此次请求
    byte[] random = this.getServiceManager().generateIdArray();
    
    // 执行Redis Lua脚本进行令牌获取逻辑
    return this.commandExecutor.evalWriteAsync(
        this.getRawName(),       // 限流器的名称
        LongCodec.INSTANCE,      // 编解码器
        command,                 // Redis命令
        
        // Lua脚本内容 (我会分段解释)
        "local rate = redis.call('hget', KEYS[1], 'rate');" +
        "local interval = redis.call('hget', KEYS[1], 'interval');" +
        "local type = redis.call('hget', KEYS[1], 'type');" +
        // 确保限流器已初始化
        "assert(rate ~= false and interval ~= false and type ~= false, 'RateLimiter is not initialized')" +
        
        // 根据限流类型选择不同的键（全局模式或每客户端模式）
        "local valueName = KEYS[2];" +
        "local permitsName = KEYS[4];" +
        "if type == '1' then " +
            "valueName = KEYS[3];" +
            "permitsName = KEYS[5];" +
        "end;" +
        
        // 确保请求的令牌数不超过定义的速率
        "assert(tonumber(rate) >= tonumber(ARGV[1]), 'Requested permits amount cannot exceed defined rate');" +
        
        // 获取当前可用令牌数量
        "local currentValue = redis.call('get', valueName);" +
        "local res;" +
        
        // 如果已有令牌计数器存在
        "if currentValue ~= false then " +
            // 清理过期的令牌记录并释放这些令牌
            "local expiredValues = redis.call('zrangebyscore', permitsName, 0, tonumber(ARGV[2]) - interval);" +
            "local released = 0;" +
            "for i, v in ipairs(expiredValues) do " +
                "local random, permits = struct.unpack('Bc0I', v);" +
                "released = released + permits;" +
            "end;" +
            
            // 如果有过期令牌被释放
            "if released > 0 then " +
                // 移除过期的令牌记录
                "redis.call('zremrangebyscore', permitsName, 0, tonumber(ARGV[2]) - interval);" +
                
                // 更新可用令牌数，确保不超过最大速率
                "if tonumber(currentValue) + released > tonumber(rate) then " +
                    "local values = redis.call('zrange', permitsName, 0, -1);" +
                    "local used = 0;" +
                    "for i, v in ipairs(values) do " +
                        "local random, permits = struct.unpack('Bc0I', v);" +
                        "used = used + permits;" +
                    "end;" +
                    "currentValue = tonumber(rate) - used;" +
                "else " +
                    "currentValue = tonumber(currentValue) + released;" +
                "end;" +
                "redis.call('set', valueName, currentValue);" +
            "end;" +
            
            // 判断是否有足够的令牌可用
            "if tonumber(currentValue) < tonumber(ARGV[1]) then " +
                // 令牌不足，计算需要等待的时间
                "local firstValue = redis.call('zrange', permitsName, 0, 0, 'withscores');" +
                "res = 3 + interval - (tonumber(ARGV[2]) - tonumber(firstValue[2]));" +
            "else " +
                // 令牌充足，记录本次获取，并减少可用令牌
                "redis.call('zadd', permitsName, ARGV[2], struct.pack('Bc0I', string.len(ARGV[3]), ARGV[3], ARGV[1]));" +
                "redis.call('decrby', valueName, ARGV[1]);" +
                "res = nil;" +
            "end;" +
        "else " +
            // 如果是首次使用，初始化计数器
            "redis.call('set', valueName, rate);" +
            "redis.call('zadd', permitsName, ARGV[2], struct.pack('Bc0I', string.len(ARGV[3]), ARGV[3], ARGV[1]));" +
            "redis.call('decrby', valueName, ARGV[1]);" +
            "res = nil;" +
        "end;" +
        
        // 同步过期时间
        "local ttl = redis.call('pttl', KEYS[1]);" +
        "if ttl > 0 then " +
            "redis.call('pexpire', valueName, ttl);" +
            "redis.call('pexpire', permitsName, ttl);" +
        "end;" +
        "return res;",
        
        // 参数列表
        Arrays.asList(
            this.getRawName(),          // KEYS[1]: 限流器名称
            this.getValueName(),        // KEYS[2]: 全局模式下的值名称
            this.getClientValueName(),  // KEYS[3]: 客户端模式下的值名称
            this.getPermitsName(),      // KEYS[4]: 全局模式下的权限名称
            this.getClientPermitsName() // KEYS[5]: 客户端模式下的权限名称
        ),
        new Object[]{
            value,                    // ARGV[1]: 请求的令牌数量
            System.currentTimeMillis(), // ARGV[2]: 当前时间戳
            random                    // ARGV[3]: 随机ID
        }
    );
}
```

